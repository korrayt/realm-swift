# Migrating from Realm Swift 10 with Atlas Device Sync

Realm Swift 20 is a local-database-only SDK. Atlas Device Sync, Atlas App
Services authentication, Functions, and the remote MongoDB client APIs are not
included.

This repository follows the upstream `community` branch. Applications that
only use local Realm files can upgrade after validating their schema
migrations and supported toolchain versions. Applications that use Atlas
Device Sync require an application-level migration.

## Removed API areas

Code using the following concepts must be redesigned before upgrading:

- `App`, `User`, `Credentials`, and App Services authentication
- partition-based and Flexible Sync configurations
- `SyncSession`, subscriptions, client reset, and sync progress APIs
- Atlas Functions and remote MongoDB collection APIs
- asymmetric objects and server-only sync models
- SwiftUI helpers that open or observe synchronized Realms

## Recommended migration sequence

1. Pin production applications to the last compatible Realm Swift 10 release
   while the migration is developed.
2. Inventory every use of the removed APIs and classify it as authentication,
   synchronization, remote data access, or local persistence.
3. Export or download all server-backed data required on the device before
   disabling the old sync path. A local Realm file is not a substitute for a
   verified server-side backup.
4. Replace authentication and remote APIs with the application's chosen
   backend. Keep this network layer separate from Realm models.
5. Open a normal local Realm with `Realm.Configuration`, then write validated
   backend responses in explicit Realm write transactions.
6. Implement an application-owned outbox, retry, conflict-resolution, and
   deletion policy if bidirectional synchronization is still required.
7. Test upgrades using copies of real Realm files, including encryption keys,
   schema-version changes, migrations, offline startup, and interrupted
   network requests.
8. Release the migration gradually and retain a rollback/export path until the
   new backend flow is verified in production.

## Local-only replacement

Applications that do not need server synchronization can remove the App
Services setup and open Realm directly:

```swift
let configuration = Realm.Configuration(
    schemaVersion: 1,
    migrationBlock: { migration, oldSchemaVersion in
        if oldSchemaVersion < 1 {
            // Apply application-specific local schema changes.
        }
    }
)

let realm = try Realm(configuration: configuration)
```

Do not copy an old synchronized Realm file into the local-only flow without
testing it. Prefer an explicit export/import process so ownership, primary
keys, pending writes, and server-only fields are handled intentionally.

## Validation checklist

- The package resolves Realm Swift 20 and Realm Core 20.
- No application source imports or references removed Sync/App Services APIs.
- Local schema migrations pass against representative production files.
- Offline create, read, update, delete, and relaunch behavior is verified.
- Authentication and remote-data failures do not corrupt local Realm writes.
- Retry and conflict behavior is deterministic and observable.
- A server-side backup and a tested rollback/export procedure exist.

