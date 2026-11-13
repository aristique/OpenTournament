# Сохранение входа в OpenTournament: исследование вариантов

Дата проверки: 2026-11-13. Статус: техническое предложение; реализации нет.
Пользователь согласовал только UX: вход сохраняется после обновления страницы. Срок сессии, способ хранения и замена прежнего JWT-подхода требуют отражения как технических решений, а не пользовательских требований.

## Вывод

Предлагается непрозрачная серверная сессия в существующем `identity-service`: случайный идентификатор в HttpOnly cookie, запись сессии в собственной PostgreSQL БД identity. Это проектный выбор для учебной недели: один вид учётных данных и один механизм отзыва вместо access/refresh-пары. Серверное хранение и серверное завершение сессий соответствуют [OWASP Session Management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html).

Цена выбора — каждый защищённый запрос бизнес-сервиса синхронно зависит от identity и его БД. Это сознательная замена прежней локальной проверки JWT: офлайн-проверка подписи и оперативный централизованный отзыв требуют разных механизмов. Различие самодостаточных токенов и серверных дескрипторов объясняет [RFC 7009, §3](https://www.rfc-editor.org/rfc/rfc7009.html#section-3).

## Сравнение

| Вопрос | Непрозрачная cookie-сессия — предложение | Refresh cookie + access JWT в памяти |
| --- | --- | --- |
| Refresh страницы | Cookie остаётся; UI получает пользователя через `/auth/session` | UI должен сначала обменять refresh cookie на новый access JWT |
| Состояние identity | Одна таблица сессий | Состояние refresh-токенов, ротация/отзыв; дополнительно выпуск и проверка JWT |
| Проверка бизнес-запроса | Вызов identity; пользователь определяется сервером | Локальная подпись/срок JWT, если не добавлена онлайн-проверка |
| Logout | Отзыв одной серверной записи | Отзыв refresh не отзывает уже выданный офлайн-проверяемый access JWT |
| Отказ identity | Защищённые запросы временно недоступны | Уже выданные JWT могут работать до истечения срока |

Таблица — вывод для этих двух предложенных схем, не универсальное сравнение библиотек. Непосредственно отсутствие немедленного отзыва access при отзыве refresh отмечено в [RFC 7009, §5](https://www.rfc-editor.org/rfc/rfc7009.html#section-5). При прежних 30 минутах access JWT возможное окно после logout составляет оставшийся срок JWT; меньший срок сокращает окно, но не устраняет его.

## Предлагаемый контракт сессии

1. Успешный `POST /auth/login` создаёт новый случайный секрет из 32 криптографически случайных байтов; браузер получает только непрозрачное значение. Новый идентификатор после аутентификации предотвращает повторное использование предшествующей сессии; требования к случайности и обновлению описаны в [OWASP Session Management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html).
2. Проектная таблица identity: `session_hash`, `user_id`, `csrf_token`, `created_at`, `expires_at`, `revoked_at`; `last_seen_at` нужен только при выборе idle-политики. Хранение SHA-256 высокоэнтропийного секрета вместо исходного bearer-секрета — наше техническое предложение; исходное значение не писать в логи. Общие требования к защите хранилища и обращению с session ID: [OWASP Session Management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html).
3. После сравнения в ТЗ предложен абсолютный срок 8 часов без скользящего продления. Рассматривался дополнительный 30-минутный idle timeout, но он не выбран в базовый контракт: это ещё одно правило и запись активности, не необходимые для принятого refresh-UX. Срок — настраиваемое техническое предложение, не отдельное решение пользователя. Cookie без `Max-Age`/`Expires` достаточно для refresh страницы; сохранение после закрытия браузера не обещается. Сессионные cookie и восстановление браузерной сессии описаны в [MDN Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie).
4. На старте и после refresh UI запрашивает `GET /auth/session`; ответ: текущий пользователь и CSRF token либо `401`. Предложение: `Cache-Control: no-store`; UI держит пользователя и CSRF token в памяти. Bearer-секрет остаётся только в cookie; `localStorage`/`sessionStorage` для него не использовать: [OWASP HTML5 Security](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html).

## Cookie и единый браузерный origin

Проектная конфигурация HTTPS: `session=<secret>; Path=/; HttpOnly; Secure; SameSite=Lax`, без `Domain`. `HttpOnly` запрещает чтение cookie через JavaScript, но браузер продолжает отправлять её в запросах; это не устраняет CSRF или XSS. Назначение атрибутов подтверждает [MDN Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie).

Frontend и `/api/*` обслуживаются через один gateway/reverse proxy и один origin (схема, хост, порт). Для разработки на локальном HTTP явно допускается `Secure=false` только в dev-конфигурации; при HTTPS — `true`. Не полагаться на одинаковую поддержку localhost-исключений браузерами: различие отмечает [MDN Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies). Это конфигурация существующего входа, не седьмой FastAPI сервис.

FastAPI умеет выставлять cookie через `Response.set_cookie`: [FastAPI Response Cookies](https://fastapi.tiangolo.com/advanced/response-cookies/). Не путать это решение со стандартным `SessionMiddleware`: Starlette описывает его как подписанную cookie с читаемым содержимым; сам по себе он не даёт таблицу отзываемых серверных сессий: [Starlette Middleware](https://www.starlette.io/middleware/#sessionmiddleware).

## Проверка в каждом сервисе

Предложение: общая небольшая FastAPI dependency защищённого пользовательского API читает session cookie и вызывает закрытый `POST /internal/sessions/validate` identity. Identity проверяет хеш, отзыв, срок и возвращает `user_id`; для изменяющей операции также сверяет CSRF token. Повторный результат не кешируется между запросами. Это не требование браузерной cookie от фонового процесса: внутренние команды отдельно проверяют служебного вызывающего и разрешённую операцию. Интроспекция и риск устаревшего кеша описаны в [RFC 7662, §4](https://www.rfc-editor.org/rfc/rfc7662.html#section-4); это аналогичный внутренний контракт, не заявление о полной реализации OAuth.

Предложение для локальной лабораторной: endpoint доступен только во внутренней сети, не маршрутизируется gateway, требует отдельный секрет вызывающего сервиса из окружения. Identity принимает секрет сессии только в теле/заголовке, не в URL. При переходе за пределы локальной среды требуется защищённый транспорт. RFC требует защищать introspection endpoint от неавторизованных запросов: [RFC 7662, §2.1](https://www.rfc-editor.org/rfc/rfc7662.html#section-2.1).

Бизнес-сервис сам проверяет право на конкретный ресурс: например, капитанство команды или владение турниром. Заголовок `X-User-ID` от браузера не является доказательством личности; gateway его удаляет, сервис определяет пользователя через identity. Пропагация личности и локальная проверка бизнес-прав разграничены в [OWASP Microservices Security](https://cheatsheetseries.owasp.org/cheatsheets/Microservices_Security_Cheat_Sheet.html).

Предложение при недоступном identity/БД: защищённая операция возвращает `503` и ничего не изменяет; не подменять сбой проверки ответом «сессия истекла». Общая зависимость поддерживается штатным механизмом [FastAPI Dependencies](https://fastapi.tiangolo.com/tutorial/dependencies/). Отдельные БД и роли сохраняются: бизнес-сервисы не читают таблицу identity напрямую; Redis, внешний IdP и новый сервис не нужны.

## CSRF, включая вход и выход

Проектный минимум для `POST`/`PUT`/`PATCH`/`DELETE`: точная проверка `Origin` по настроенному origin приложения; если заголовок отсутствует — проверка origin из `Referer`, иначе отказ. Не разрешать `null` и не сравнивать строковым префиксом. Дополнительно нужен случайный CSRF token сессии в `X-CSRF-Token`; сервер сравнивает его с записью сессии. Для stateful приложений synchronizer token рекомендован [OWASP CSRF Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html).

Login/registration ещё не имеют authenticated session: предложение — JSON-only, обязательный пользовательский заголовок `X-CSRF-Protection: 1` и та же проверка Origin/Referer; без разрешающего CORS для чужих origin. Logout — только `POST` с session CSRF token. `GET` не выполняет бизнес-изменений; `SameSite=Lax` остаётся дополнительным барьером, а не единственной защитой. Custom headers, login CSRF и ограничения SameSite разобраны в [OWASP CSRF Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html).

## Точная гарантия logout и проверка будущей реализации

Предложение: `POST /auth/logout` сначала фиксирует `revoked_at` в БД identity, затем возвращает успех и удаляет cookie с теми же именем/Path/Domain. UI очищает своё состояние после успешного ответа. При сбое отзыва нельзя показывать подтверждённый серверный выход. Необходимость серверной инвалидизации подчёркивает [OWASP Logout Testing](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/06-Session_Management_Testing/06-Testing_for_Logout_Functionality).

Гарантия предлагаемой схемы без кеша: проверка сессии, начатая после фиксации отзыва, получает отказ. Запрос, уже прошедший проверку до logout, может завершиться позже; завершённые действия и уже принятые RabbitMQ задания logout не отменяет. Это граница выбранного контракта, а не обещание атомарности между шестью БД. Более строгая сериализация потребовала бы дополнительного протокола и не предлагается для этой лабораторной.

Будущие приёмочные проверки: refresh сохраняет пользователя; повтор украденной старой cookie после успешного logout даёт `401`; неверный Origin/CSRF даёт `403`; чужой ресурс запрещён даже при действующей сессии; отказ identity даёт `503`; истёкшая сессия даёт `401`; конкурентный запрос демонстрирует описанную границу logout. Это предлагаемые проверки проекта, сейчас они не выполнялись.
