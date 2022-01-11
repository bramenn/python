## v0.14.3 (2022-01-05)

* *(types)* Added `MatrixURI` type to parse and build `matrix:` URIs and
  `https://matrix.to` URLs.
* *(util.formatter)* `matrix:` URIs are now supported in incoming messages
  (using the new parser mentioned above).
* *(util.variation_selector)* Switched to generating list of emoji using data
  directly from the Unicode spec instead of emojibase.
* *(util.formatter)* Whitespace in non-`pre` elements is now compressed into
  a single space. Newlines are also replaced with a space instead of removed
  completely. Whitespace after a block element is removed completely.
* *(util.ffmpeg)* Added option to override output path, which allows outputting
  to stdout (by specifying `-`).
* *(util.config)* Changed `ConfigUpdateHelper.copy` to ignore comments if the
  entity being copied is a commentable yaml object (e.g. map or list).

## v0.14.2 (2021-12-30)

* *(appservice)* Fixed `IntentAPI` throwing an error when `redact` was called
  with a `reason`, but without `extra_content`.

## v0.14.1 (2021-12-29)

* *(util.ffmpeg)* Added simple utility module that wraps ffmpeg and tempfiles
  to convert audio/video files to different formats, primarily intended for
  bridging. FFmpeg must be installed separately and be present in `$PATH`.

## v0.14.0 (2021-12-26)

* **Breaking change *(mautrix.util.formatter)*** Made `MatrixParser` async
  and non-static.
  * Being async is necessary for bridges that need to make database calls to
    convert mentions (e.g. Telegram has @username mentions, which can't be
    extracted from the Matrix user ID).
  * Being non-static allows passing additional context into the formatter by
    extending the class and setting instance variables.
* *(util.formatter)* Added support for parsing
  [spoilers](https://spec.matrix.org/v1.1/client-server-api/#spoiler-messages).
* *(crypto.olm)* Added `describe` method for `OlmSession`s.
* *(crypto)* Fixed sorting Olm sessions (now sorted by last successful decrypt
  time instead of alphabetically by session ID).
* *(crypto.store.asyncpg)* Fixed caching Olm sessions so that using the same
  session twice wouldn't cause corruption.
* *(crypto.attachments)* Added support for decrypting files from
  non-spec-compliant clients (e.g. FluffyChat) that have a non-zero counter
  part in the AES initialization vector.
* *(util.async_db)* Added support for using Postgres positional param syntax in
  the async SQLite helper (by regex-replacing `$<number>` with `?<number>`).
* *(util.async_db)* Added wrapper methods for `executemany` in `Database` and
  aiosqlite `TxnConnection`.
* *(bridge)* Changed portal cleanup to leave and forget rooms using double
  puppeting instead of just kicking the user.

## v0.13.3 (2021-12-15)

* Fixed type hints in the `mautrix.crypto.store` module.
* Added debug logs for detecting crypto sync handling slowness.

## v0.13.2 (2021-12-15)

* Switched message double puppet indicator convention from
  `"net.maunium.<bridge_type>.puppet": true` to `"fi.mau.double_puppet_source": "<bridge_type>"`.
* Added double puppet indicator to redactions made with `IntentAPI.redact`.

## v0.13.1 (2021-12-12)

* Changed lack of media encryption dependencies (pycryptodome) to be a fatal
  error like lack of normal encryption dependencies (olm) are in v0.13.0.
* Added base methods for implementing relay mode in bridges
  (started by [@Alejo0290] in [#72]).

[@Alejo0290]: https://github.com/Alejo0290
[#72]: https://github.com/mautrix/python/pull/72

## v0.13.0 (2021-12-09)

* Formatted all code using [black](https://github.com/psf/black)
  and [isort](https://github.com/PyCQA/isort).
* Added `power_level_override` parameter to `ClientAPI.create_room`.
* Added default implementations of `delete-portal` and `unbridge` commands for bridges
* Added automatic Olm session recreation if an incoming message fails to decrypt.
* Added automatic key re-requests in bridges if the Megolm session doesn't arrive on time.
* Changed `ClientAPI.send_text` to parse the HTML to generate a plaintext body
  instead of using the HTML directly when a separate plaintext body is not
  provided (also affects `send_notice` and `send_emote`).
* Changed lack of encryption dependencies to be a fatal error if encryption is
  enabled in bridge config.
* Fixed `StoreUpdatingAPI` not updating the local state store when using
  friendly membership methods like `kick_user`.
* Switched Bridge class to use async_db (asyncpg/aiosqlite) instead of the
  legacy SQLAlchemy db by default.
* Removed deprecated `ClientAPI.parse_mxid` method
  (use `ClientAPI.parse_user_id` instead).
* Renamed `ClientAPI.get_room_alias` to `ClientAPI.resolve_room_alias`.

## v0.12.5 (2021-11-30)

* Added wrapper for [MSC2716]'s `/batch_send` endpoint in `IntentAPI`
* Added some Matrix request metrics (thanks to @jaller94 in #68)
* Added utility method for adding variation selector 16 to emoji strings the
  same way as Element does (using emojibase data).

[MSC2716]: https://github.com/matrix-org/matrix-doc/pull/2716

## v0.12.4 (2021-11-25)

* *(util.formatter)* Added support for parsing Matrix HTML colors.

## v0.12.3 (2021-11-23)

* Added autogenerated docs with Sphinx.
  * Rendered version available at https://docs.mau.fi/python/latest/
    (also version-specific docs at https://docs.mau.fi/python/v0.12.3/).
* Added asyncpg to client state store unit tests.
* Fixed client state store `get_members` being broken on asyncpg (broken in 0.12.2).
* Fixed `get_members_filtered` not taking the `memberships` parameter into
  account in the memory store.

## v0.12.2 (2021-11-20)

* Added more control over which membership states to return in client state store.
* Added some basic tests for the client state store.
* Fixed `OlmMachine.account` property not being defined before calling `load`.

## v0.12.1 (2021-11-19)

* Added default (empty) value for `unsigned` in the event classes.
* Updated the `PgStateStore` in the client module to fully implement the crypto
  `StateStore` abstract class.
  * The crypto module now has a `PgCryptoStateStore` that combines the client
    `PgStateStore` with the abstract crypto state store.

## v0.12.0 (2021-11-19)

* **Breaking change (client):** The `whoami` method now returns a dataclass
  with `user_id` and `device_id` fields, instead of just returning the
  `user_id` as a string.
* Added `delete` method for crypto stores (useful when changing the device ID).
* Added `DECRYPTED` step for message send checkpoints.
* Added proper user agent to bridge state and message send checkpoint requests.

## v0.11.4 (2021-11-16)

* Improved default event filter in bridges
  * The filtering method is now `allow_matrix_event` instead of
    `filter_matrix_event` and the return value is reversed.
  * Most bridges now don't need to override the method, so the old method isn't
    used at all.
* Added support for the stable version of [MSC2778].

## v0.11.3 (2021-11-13)

* Updated registering appservice ghosts to use `inhibit_login` flag to prevent
  lots of unnecessary access tokens from being created.
  * If you want to log in as an appservice ghost, you should use [MSC2778]'s
    appservice login (e.g. like the [bridge e2ee module does](https://github.com/mautrix/python/blob/v0.11.2/mautrix/bridge/e2ee.py#L178-L182) for example)
* Fixed unnecessary warnings about message send endpoints in some cases where
  the endpoint wasn't configured.

## v0.11.2 (2021-11-11)

* Updated message send checkpoint system to handle all cases where messages are
  dropped or consumed by mautrix-python.

## v0.11.1 (2021-11-10)

* Fixed regression in Python 3.8 support in v0.11.0 due to `asyncio.Queue` type hinting.
* Made the limit of HTTP connections to the homeserver configurable
  (thanks to [@justinbot] in [#64]).

[#64]: https://github.com/mautrix/python/pull/64

## v0.11.0 (2021-11-09)

* Added support for message send checkpoints (as HTTP requests, similar to the
  bridge state reporting system) by [@sumnerevans].
* Added support for aiosqlite with the same interface as asyncpg.
  * This includes some minor breaking changes to the asyncpg interface.
* Made config writing atomic (using a tempfile) to prevent the config
  disappearing when disk is full.
* Changed prometheus to start before rest of `startup_actions`
  (thanks to [@Half-Shot] in [#63]).
* Stopped reporting `STARTING` bridge state on startup by [@sumnerevans].

[@Half-Shot]: https://github.com/Half-Shot
[#63]: https://github.com/mautrix/python/pull/63

## v0.10.11 (2021-10-26)

* Added support for custom bridge bot welcome messages
  (thanks to [@justinbot] in [#58]).

[@justinbot]: https://github.com/justinbot
[#58]: https://github.com/mautrix/python/pull/58

## v0.10.10 (2021-10-08)

* Added support for disabling bridge management commands based on custom rules
  (thanks to [@tadzik] in [#56]).

[@tadzik]: https://github.com/tadzik
[#56]: https://github.com/mautrix/python/pull/56

## v0.10.9 (2021-09-29)

* Changed `remove_room_alias` to ignore `M_NOT_FOUND` errors by default, to
  preserve Synapse behavior on spec-compliant server implementations.
  The `raise_404` argument can be set to `True` to not suppress the errors.
* Fixed bridge state pings returning `UNCONFIGURED` as a global state event.

## v0.10.8 (2021-09-23)

* **Breaking change (serialization):** Removed `Generic[T]` backwards
  compatibility from `SerializableAttrs` (announced in [v0.9.6](https://github.com/mautrix/python/releases/tag/v0.9.6)).
* Stopped using `self.log` in `Program` config load errors as the logger won't
  be initialized yet.
* Added check to ensure reply fallback removal is only attempted once.
* Fixed `remove_event_handler` throwing a `KeyError` if no event handlers had
  been registered for the specified event type.
* Fixed deserialization showing wrong key names on missing key errors.

## v0.10.7 (2021-08-31)

* Removed Python 3.9+ features that were accidentally used in v0.10.6.

## v0.10.6 (2021-08-30)

* Split `_http_handle_transaction` in `AppServiceServerMixin` to allow easier reuse.

## v0.10.5 (2021-08-25)

* Fixed `MemoryStateStore`'s `get_members()` implementation (thanks to [@hifi] in [#54]).
* Re-added `/_matrix/app/com.beeper.bridge_state` endpoint.

[@hifi]: https://github.com/hifi
[#54]: https://github.com/mautrix/python/pull/54

## v0.10.4 (2021-08-18)

* Improved support for sending member events manually
  (when using the `extra_content` field in join, invite, etc).
  * There's now a `fill_member_event` method that's called by manual member
    event sending that adds the displayname and avatar URL. Alternatively,
    `fill_member_event_callback` can be set to fill the member event manually.

## v0.10.3 (2021-08-14)

* **Breaking change:** The bridge status notification system now uses a
  `BridgeStateEvent` enum instead of the `ok` boolean.
* Added better log messages when bridge encryption error notice fails to send.
* Added manhole for all bridges.
* Dropped Python 3.6 support in manhole.
* Switched to using `PyCF_ALLOW_TOP_LEVEL_AWAIT` for manhole in Python 3.8+.

## v0.9.10 (2021-07-24)

* Fixed async `Database` class mutating the `db_args` dict passed to it.
* Fixed `None`/`null` values with factory defaults being deserialized into the
  `attr.Factory` object instead of the expected value.

## v0.9.9 (2021-07-16)

* **Breaking change:** Made the `is_direct` property required in the bridge
  `Portal` class. The property was first added in v0.8.4 and is used for
  handling `m.room.encryption` events (enabling encryption).
* Added PEP 561 typing info (by [@sumnerevans] in [#49]).
* Added support for [MSC3202] in appservice module.
* Made bridge state filling more customizable.
* Moved `BridgeState` class from `mautrix.bridge` to `mautrix.util.bridge_state`.
* Fixed receiving appservice transactions with `Authorization` header
  (i.e. fixed [MSC2832] support).

[MSC3202]: https://github.com/matrix-org/matrix-doc/pull/3202
[MSC2832]: https://github.com/matrix-org/matrix-doc/pull/2832
[@sumnerevans]: https://github.com/sumnerevans
[#49]: https://github.com/mautrix/python/pull/49

## v0.9.8 (2021-06-24)

* Added `remote_id` field to `push_bridge_state` method.

## v0.9.7 (2021-06-22)

* Added tests for `factory` and `hidden` serializable attrs.
* Added `login-matrix`, `logout-matrix`, `ping-matrix` and `clear-cache-matrix`
  commands in the bridge module. To enable the commands, bridges must implement
  the `User.get_puppet()` method to return the `Puppet` instance corresponding
  to the user's remote ID.
* Fixed logging events that were ignored due to lack of permissions of the sender.
* Fixed deserializing encrypted edit events ([mautrix/telegram#623]).

[mautrix/telegram#623]: https://github.com/mautrix/telegram/issues/623

## v0.9.6 (2021-06-20)

* Replaced `GenericSerializable` with a bound `TypeVar`.
  * This means that classes extending `SerializableAttrs` no longer have to use
    the `class Foo(SerializableAttrs['Foo'])` syntax to get type hints, just
    `class Foo(SerializableAttrs)` is enough.
  * Backwards compatibility for using the `['Foo']` syntax will be kept until v0.10.
* Added `field()` as a wrapper for `attr.ib()` that makes it easier to add
  custom metadata for serializable attrs things.
* Added some tests for type utilities.
* Changed attribute used to exclude links from output in HTML parser.
  * New attribute is `data-mautrix-exclude-plaintext` and works for basic
    formatting (e.g. `<strong>`) in addition to `<a>`.
  * The previous attribute wasn't actually checked correctly, so it never worked.

## v0.9.5 (2021-06-11)

* Added `SynapseAdminPath` to build `/_synapse/admin` paths.

## v0.9.4 (2021-06-09)

* Updated bridge status pushing utility to support `remote_id` and `remote_name`
  fields to specify which account on the remote network is bridged.

## v0.9.3 (2021-06-04)

* Switched to stable space prefixes.
* Added option to send arbitrary content with membership events.
* Added warning if media encryption dependencies aren't installed.
* Added support for pycryptodomex for media encryption.
* Added utilities for pushing bridge status to an arbitrary HTTP endpoint.

## v0.9.2 (2021-04-26)

* Changed `update_direct_chats` bridge method to only send updated `m.direct`
  data if the content was modified.

## v0.9.1 (2021-04-20)

* Added type classes for VoIP.
* Added methods for modifying push rules and room tags.
* Switched to `asyncio.create_task` everywhere (replacing the older
  `loop.create_task` and `asyncio.ensure_future`).

## v0.9.0 (2021-04-16)

* Added option to retry all HTTP requests when encountering a HTTP network
  error or gateway error response (502/503/504) 
  * Disabled by default, you need to set the `default_retry_count` field in
    `HTTPAPI` (or `Client`), or the `default_http_retry_count` field in
    `AppService` to enable.
  * Can also be enabled with `HTTPAPI.request()`s `retry_count` parameter.
  * The `mautrix.util.network_retry` module was removed as it became redundant.
* Fixed GET requests having a body ([#44]).

[#44]: https://github.com/mautrix/python/issues/44

## v0.8.18 (2021-04-01)

* Made HTTP request user agents more configurable.
  * Bridges will now include the name and version by default.
* Added some event types and classes for space events.
* Fixed local power level check failing for `m.room.member` events.

## v0.8.17 (2021-03-22)

* Added warning log when giving up on decrypting message.
* Added mimetype magic utility that supports both file-magic and python-magic.
* Updated asmux DM endpoint (`net.maunium.asmux` -> `com.beeper.asmux`).
* Moved RowProxy and ResultProxy imports into type checking ([#46]).
  This should fix SQLAlchemy 1.4+, but SQLAlchemy databases will likely be
  deprecated entirely in the future.

[#46]: https://github.com/mautrix/python/issues/46

## v0.8.16 (2021-02-16)

* Made the Bridge class automatically fetch media repo config at startup.
  Bridges are recommended to check `bridge.media_config.upload_size` before
  even downloading remote media.

## v0.8.15 (2021-02-08)

* Fixed the high-level `Client` class to not try to update state if there' no
  `state_store` set.

## v0.8.14 (2021-02-07)

* Added option to override the asyncpg pool used in the async `Database` wrapper.

## v0.8.13 (2021-02-07)

* Stopped checking error message when checking if user is not registered on
  whoami. Now it only requires the `M_FORBIDDEN` errcode instead of a specific
  human-readable error message.
* Added handling for missing `unsigned` object in membership events
  (thanks to [@jevolk] in [#39]).
* Added warning message when receiving encrypted messages with end-to-bridge
  encryption disabled.
* Added utility for mutexes in caching async getters to prevent race conditions.

[@jevolk]: https://github.com/jevolk
[#39]: https://github.com/mautrix/python/pull/39

## v0.8.12 (2021-02-01)

* Added handling for `M_NOT_FOUND` errors when getting pinned messages.
* Fixed bridge message send retrying so it always uses the same transaction ID.
* Fixed high-level `Client` class to automatically update state store with
  events from sync.

## v0.8.11 (2021-01-22)

* Added automatic login retry if double puppeting token is invalid on startup
  or gets invalidated while syncing.
* Fixed ExtensibleEnum leaking keys between different types.
* Allowed changing bot used in ensure_joined.

## v0.8.10 (2021-01-22)

* Changed attr deserialization errors to log full data instead of only known
  fields when deserialization fails.

## v0.8.9 (2021-01-21)

* Allowed `postgresql://` scheme in end-to-bridge encryption database URL
  (in addition to `postgres://`).
* Slightly improved attr deserialization error messages.

## v0.8.8 (2021-01-19)

* Changed end-to-bridge encryption to fail if homeserver doesn't advertise
  appservice login. This breaks Synapse 1.21, but there have been plenty of
  releases since then.
* Switched BaseFileConfig to use the built-in [pkgutil] instead of
  pkg_resources (which requires setuptools).
* Added handling for `M_NOT_FOUND` errors when updating `m.direct` account data
  through double puppeting in bridges.
* Added logging of data when attr deserializing fails.
* Exposed ExtensibleEnum in `mautrix.types` module.

[pkgutil]: https://docs.python.org/3/library/pkgutil.html

## v0.8.7 (2021-01-15)

* Changed attr deserializer to deserialize optional missing fields into `None`
  instead of `attr.NOTHING` by default.
* Added option not to use transaction for asyncpg database upgrades.

## v0.8.6 (2020-12-31)

* Added logging when sync errors are resolved.
* Made `.well-known` fetching ignore the response content type header.
* Added handling for users enabling encryption in private chat portals.

## v0.8.5 (2020-12-06)

* Made SerializableEnum work with int values.
* Added TraceLogger type hints to command handling classes.

## v0.8.4 (2020-12-02)

* Added logging when sync errors are resolved.
* Made `.well-known` fetching ignore the response content type header.
* Added handling for users enabling encryption in private chat portals.

## v0.8.3 (2020-11-17)

* Fixed typo in HTML reply fallback generation when target message is plaintext.
* Made `CommandEvent.mark_read` async instead of returning an awaitable,
  because sometimes it didn't return an awaitable.

## v0.8.2 (2020-11-10)

* Added utility function for retrying network calls
  (`from mautrix.util.network_retry import call_with_net_retry`).
* Updated `Portal._send_message` to use aforementioned utility function.

## v0.8.1 (2020-11-09)

* Changed `Portal._send_message` to retry after 5 seconds (up to 5 attempts
  total by default) if server returns 502/504 error or the connection fails.

## v0.8.0 (2020-11-07)

* Added support for cross-server double puppeting
  (thanks to [@ShadowJonathan] in [#26]).
* Added support for receiving ephemeral events pushed directly ([MSC2409]).
* Added `opt_prometheus` utility to add support for metrics without a hard
  dependency on the prometheus_client library.
* Added `formatted()` helper method to get the `formatted_body` of a text message.
* Bridge command system improvements
  (thanks to [@witchent] in [#29], [#30] and [#31]).
  * `CommandEvent`s now know which portal they were ran in. They also have a
    `main_intent` property that gets the portal's main intent or the bridge bot.
  * `CommandEvent.reply()` will now use the portal's main intent if the bridge
    bot is not in the room.
  * The `needs_auth` and `needs_admin` permissions are now included here
    instead of separately in each bridge.
  * Added `discard-megolm-session` command.
  * Moved `set-pl` and `clean-rooms` commands from mautrix-telegram.
* Switched to using yarl instead of manually concatenating base URL with path.
* Switched to appservice login ([MSC2778]) instead of shared secret login for
  bridge bot login in the end-to-bridge encryption helper.
* Switched to `TEXT` instead of `VARCHAR(255)` in all databases ([#28]).
* Changed replies to use a custom `net.maunium.reply` relation type instead of `m.reference`.
* Fixed potential db unique key conflicts when the membership state caches were
  updated from `get_joined_members`.
* Fixed database connection errors causing sync loops to stop completely.
* Fixed `EventType`s sometimes having `None` instead of
  `EventType.Class.UNKNOWN` as the type class.
* Fixed regex escaping in bridge registration generation.

[MSC2778]: https://github.com/matrix-org/matrix-doc/pull/2778
[MSC2409]: https://github.com/matrix-org/matrix-doc/pull/2409
[@ShadowJonathan]: https://github.com/ShadowJonathan
[@witchent]: https://github.com/witchent
[#26]: https://github.com/mautrix/python/pull/26
[#28]: https://github.com/mautrix/python/issues/28
[#29]: https://github.com/mautrix/python/pull/29
[#30]: https://github.com/mautrix/python/pull/30
[#31]: https://github.com/mautrix/python/pull/31

## v0.7.14 (2020-10-27)

* Wrapped union types in `NewType` to allow `setattr`.
  This fixes Python 3.6 and 3.9 compatibility.

## v0.7.13 (2020-10-09)

* Extended session wait time when handling encrypted messages in bridges:
  it'll now wait for 5 seconds, then send an error, then wait for 10 more
  seconds. If the keys arrive in those 10 seconds, the message is bridged
  and the error is redacted, otherwise the error is edited.

## v0.7.11 (2020-10-02)

* Lock olm sessions between encrypting and sending to make sure messages go out
  in the correct order.

## v0.7.10 (2020-09-29)

* Fixed deserializing the `info` object in media msgtypes into dataclasses.

## v0.7.9 (2020-09-28)

* Added parameter to change how long `EncryptionManager.decrypt()` should wait
  for the megolm session to arrive.
* Changed `get_displayname` and `get_avatar_url` to ignore `M_NOT_FOUND` errors.
* Updated type hint of `set_reply` to allow `EventID`s.

## v0.7.8 (2020-09-27)

* Made the `UUID` type de/serializable by default.

## v0.7.7 (2020-09-25)

* Added utility method for waiting for incoming group sessions in OlmMachine.
* Made end-to-bridge encryption helper wait for incoming group sessions for 3 seconds.

## v0.7.6 (2020-09-22)

* Fixed bug where parsing invite fails if `unsigned` is not set or null.
* Added trace logs when bridge module ignores messages.

## v0.7.5 (2020-09-19)

* Added utility for measuring async method time in prometheus.

## v0.7.4 (2020-09-19)

* Made `sender_device` optional in decrypted olm events.
* Added opt_prometheus utility for using prometheus as an optional dependency.
* Added Matrix event time processing metric for bridges when prometheus is installed.

## v0.7.3 (2020-09-17)

* Added support for telling the user about decryption errors in bridge module.

## v0.7.2 (2020-09-12)

* Added bridge config option to pass custom arguments to SQLAlchemy's `create_engine`.

## v0.7.1 (2020-09-09)

* Added optional automatic prometheus config to the `Program` class.

## v0.7.0 (2020-09-04)

* Added support for e2ee key sharing in `OlmMachine`
  (both sending and responding to requests).
* Added option for automatically sharing keys from bridges.
* Added account data get/set methods for `ClientAPI`.
* Added helper for bridges to update `m.direct` account data.
* Added default user ID and alias namespaces for bridge registration generation.
* Added asyncpg-based client state store implementation.
* Added filtering query parameters to `ClientAPI.get_members`.
* Changed attachment encryption methods to return `EncryptedFile` objects
  instead of dicts.
* Changed `SimpleLock` to use `asyncio.Event` instead of `asyncio.Future`.
* Made SQLAlchemy optional for bridges.
* Fixed error when profile endpoint responses are missing keys.

## v0.6.1 (2020-07-30)

* Fixed disabling notifications in many rooms at the same time.

## v0.6.0 (2020-07-27)

* Added native end-to-end encryption module.
  * Switched e2be helper to use native e2ee instead of matrix-nio.
  * Includes crypto stores based on pickle and asyncpg.
  * Added e2ee helper to high-level client module.
* Added support for getting `prev_content` from the top level in addition to `unsigned`.

## v0.5.8 (2020-07-27)

* Fixed deserializer using `attr.NOTHING` instead of `None` when there's no default value.

## v0.5.7 (2020-06-16)

* Added `alt_aliases` to canonical alias state event content
  (added in Matrix client-server spec r0.6.1).

## v0.5.6 (2020-06-15)

* Added support for adding aliases for bridge commands.

## v0.5.5 (2020-06-15)

* Added option to set default event type class in `EventType.find()`.

## v0.5.4 (2020-06-09)

* Fixed notification disabler breaking when not using double puppeting.

## v0.5.3 (2020-06-08)

* Added `NotificationDisabler` utility class for easily disabling notifications
  while a bridge backfills messages.

## v0.5.2 (2020-06-08)

* Added support for automatically calling `ensure_registered` if `whoami` says
  the bridge bot is not registered in `Bridge.wait_for_connection`.

## v0.5.1 (2020-06-05)

* Moved initializing end-to-bridge encryption to before other startup actions.

## v0.5.0 (2020-06-03)

* Added extensible enum class ([#14]).
* Added some asyncpg utilities.
* Added basic config validation support to disallow default values.
* Added matrix-nio based end-to-bridge encryption helper for bridges.
* Added option to use TLS for appservice listener.
* Added support for `Authorization` header from homeserver in appservice
  transaction handler.
* Added option to override appservice transaction handling method.
* Split `Bridge` initialization class into a more abstract `Program`.
* Split config loading.

[#14]: https://github.com/mautrix/python/issues/14

## v0.4.2 (2020-02-14)

* Added option to add custom arguments for programs based on the `Bridge` class.
* Added method for stopping a `Bridge`.
* Made `Obj` picklable.

## v0.4.1 (2020-01-07)

* Removed unfinished `enum.py`.
* Increased default config line wrapping width.
* Fixed default visibility when adding rooms and users with bridge community helper.

## v0.4.0 (2019-12-28)

* Initial "stable" release of the major restructuring.
  * Package now includes the Matrix client framework and other utilities
    instead of just an appservice module.
  * Package renamed from mautrix-appservice to mautrix.
  * Switched license from MIT to MPLv2.

## v0.3.11 (2019-06-20)

* Update state store after sending state event. This is required for some
  servers like t2bot.io that have disabled echoing state events to appservices.

## v0.3.10.dev1 (2019-05-23)

* Hacky fix for null `m.relates_to`'s.

## v0.3.9 (2019-05-11)

* Only use json.dumps() in request() if content is json-serializable.

## v0.3.8 (2019-02-13)

* Added missing room/event ID quotings.

## v0.3.7 (2018-09-28)

* Fixed `get_room_members()` returning `dict_keys` rather than `list` when
  getting only joined members.

## v0.3.6 (2018-08-06

* Fixed `get_room_joined_memberships()` (thanks to [@turt2live] in [#6]).

[@turt2live]: https://github.com/turt2live
[#6]: https://github.com/mautrix/python/pull/6

## v0.3.5 (2018-08-06)

* Added parameter to change aiohttp Application parameters.
* Fixed `get_power_levels()` with state store implementations that don't throw
  a `ValueError` on cache miss.

## v0.3.4 (2018-08-05)

* Updated `get_room_members()` to use `/joined_members` instead of `/members`
  when possible.

## v0.3.3 (2018-07-25)

* Updated some type hints.

## v0.3.2 (2018-07-23)

* Fixed HTTPAPI init for real users.
* Fixed content-type for empty objects.

## v0.3.1 (2018-07-22)

* Added support for real users.

## v0.3.0 (2018-07-10)

* Made `StateStore` into an abstract class for easier custom storage backends.
* Fixed response of `/transaction` to return empty object with 200 OK's as per spec.
* Fixed URL parameter encoding.
* Exported `IntentAPI` for type hinting.

## v0.2.0 (2018-06-24)

* Switched to GPLv3 to MIT license.
* Updated state store to store full member events rather than just the
  membership status.

## v0.1.5 (2018-05-06)

* Made room avatar in `set_room_avatar()` optional to allow unsetting avatar.

## v0.1.4 (2018-04-26)

* Added `send_sticker()`.

## v0.1.3 (2018-03-29)

* Fixed AppService log parameter type hint.
* Fixed timestamp handling.

## v0.1.2 (2018-03-29)

* Return 400 Bad Request if user/room query doesn't have user ID/alias field (respectively).
* Added support for timestamp massaging and source URLs.

## v0.1.1 (2018-03-11)

* Added type hints.
* Added power level checks to `set_state_event()`.
* Renamed repo to mautrix-appservice-python (PyPI package is still mautrix-appservice).

## v0.1.0 (2018-03-08)

* Initial version. Transferred from mautrix-telegram.