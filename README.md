# A test case for flutter drive not respecting pubspec.lock

`my_app` has a locked/overridden unpublished package dependency from path
similarly how [`melos`](https://github.com/invertase/melos) works.

my_app/pubspec.lock:
```
  my_package:
    dependency: "direct main"
    description:
      path: "../my_package"
      relative: true
    source: path
    version: "0.0.1"
```

## Flutter drive doesn't work

`flutter drive --no-pub` ignores `pubspec.lock`, and insists verifying that the
package has been published.

Steps to reproduce:

```
cd my_app
flutter drive --no-pub --verbose --driver=test_driver/integration_test.dart --target=integration_test/my_app_test.dart
```

Expected result: the integration test runs

Actual result:
`flutter drive --no-pub` runs `flutter pub get` and running the integration test
fails because:

> Because my_app depends on my_package any which doesn't exist (could not find package my_package at https://pub.dartlang.org), version solving failed.

<details><summary>Verbose output</summary>

```
[+4001 ms] Using /home/jpnurmi/Flutter/.pub-cache for the pub cache.
[        ] executing: [/home/jpnurmi/Temp/flutter_drive_vs_pubspec_lock/my_app/] /home/jpnurmi/Flutter/bin/cache/dart-sdk/bin/pub --verbose get --no-precompile
[  +34 ms] FINE: Pub 2.13.0-52.0.dev
[  +78 ms] MSG : Resolving dependencies...
[  +46 ms] SLVR: fact: my_app is 0.0.0
[   +6 ms] SLVR: derived: my_app
[  +21 ms] SLVR: fact: my_app depends on flutter any from sdk
[        ] SLVR: fact: my_app depends on my_package ^0.0.1
[        ] SLVR: fact: my_app depends on flutter_test any from sdk
[        ] SLVR: fact: my_app depends on integration_test any from sdk
[        ] SLVR:   selecting my_app
[        ] SLVR:   derived: integration_test any from sdk
[        ] SLVR:   derived: flutter_test any from sdk
[        ] SLVR:   derived: my_package ^0.0.1
[        ] SLVR:   derived: flutter any from sdk
[   +8 ms] IO  : Get versions from https://pub.dartlang.org/api/packages/my_package.
[  +24 ms] IO  : HTTP GET https://pub.dartlang.org/api/packages/my_package
[        ]     | Accept: application/vnd.pub.v2+json
[        ]     | X-Pub-OS: linux
[        ]     | X-Pub-Command: get
[        ]     | X-Pub-Session-ID: 6ED1FB93-E966-4F49-B479-7B737718D9ED
[        ]     | X-Pub-Environment: flutter_cli:verify:drive
[        ]     | X-Pub-Reason: direct
[        ]     | user-agent: Dart pub 2.13.0-52.0.dev
[ +379 ms] IO  : HTTP response 404 Not Found for GET https://pub.dartlang.org/api/packages/my_package
[        ]     | took 0:00:00.379739
[        ]     | transfer-encoding: chunked
[        ]     | date: Sat, 20 Feb 2021 10:08:16 GMT
[        ]     | content-encoding: gzip
[        ]     | vary: Accept-Encoding
[        ]     | strict-transport-security: max-age=31536000; preload
[        ]     | via: 1.1 google
[        ]     | content-type: application/json; charset="utf-8"
[        ]     | x-frame-options: SAMEORIGIN
[        ]     | x-xss-protection: 1; mode=block
[        ]     | x-content-type-options: nosniff
[        ]     | server: dart:io with Shelf
[  +20 ms] SLVR:   fact: my_package doesn't exist (could not find package my_package at https://pub.dartlang.org)
[   +3 ms] SLVR:   conflict: my_package doesn't exist (could not find package my_package at https://pub.dartlang.org)
[   +2 ms] SLVR:   ! my_package any is satisfied by my_package ^0.0.1
[        ] SLVR:   ! which is caused by "my_app depends on my_package ^0.0.1"
[        ] SLVR:   ! thus: version solving failed
[   +1 ms] SLVR: Version solving took 0:00:00.477964 seconds.
[        ]     | Tried 1 solutions.
[        ] FINE: Resolving dependencies finished (0.522s).
[  +13 ms] ERR : Because my_app depends on my_package any which doesn't exist (could not find package my_package at https://pub.dartlang.org), version solving failed.
[        ] FINE: Exception type: SolveFailure
[  +31 ms] FINE: package:pub/src/solver/version_solver.dart 308:5                                    VersionSolver._resolveConflict
[        ]     | package:pub/src/solver/version_solver.dart 129:27                                   VersionSolver._propagate
[        ]     | package:pub/src/solver/version_solver.dart 93:11                                    VersionSolver.solve.<fn>
[        ]     | ===== asynchronous gap ===========================
[        ]     | dart:async                                                                          Future.catchError
[        ]     | package:pub/src/utils.dart 113:52                                                   captureErrors.wrappedCallback
[        ]     | package:stack_trace                                                                 Chain.capture
[        ]     | package:pub/src/utils.dart 126:11                                                   captureErrors
[        ]     | package:pub/src/command.dart 164:13                                                 PubCommand.run
[        ]     | package:args/command_runner.dart 196:27                                             CommandRunner.runCommand
[        ]     | package:pub/src/command_runner.dart 150:26                                          PubCommandRunner.runCommand
[        ]     | package:pub/src/command_runner.dart 138:18                                          PubCommandRunner.run
[        ]     | /b/s/w/ir/cache/builder/src/third_party/dart/third_party/pkg/pub/bin/pub.dart 9:48  main
```
</details>


## Flutter run works

For comparison, `flutter run --no-pub` respects the package lock and running the
app works:

```
cd my_app
$ flutter run --no-pub
Launching lib/main.dart on Linux in debug mode...
Building Linux application...                                           
Syncing files to device Linux...                                    71ms
[...]
```
