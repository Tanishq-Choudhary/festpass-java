# FestPass: campus event registration + Jenkins

Status: the cancel bug is fixed and all 28 tests pass. See [My submission](#my-submission)
at the end of this file for what was done and where the evidence is.

The starter below was a deliberately failing **diagnostic starter**, not a finished submission. It manages
multiple events, seat limits, student registrations, cancellations, and local
file persistence. For the **one-hour Jenkins assessment**, create the SCM job,
add the missing polling trigger, investigate test failures, and demonstrate
an automatic red build followed by an automatic green build after your fix.
Read **ASSIGNMENT.md** for exact instructions and the 100-point rubric.
All application features and tests are supplied; no new Java feature is required.
The starter Jenkinsfile intentionally has no `triggers` block.
Fill `submission.properties` with your instructor-issued class/challenge codes before
building in Jenkins. The alias comes automatically from the checked-out commit's
Git author name (`git log -1 --format=%an`), not from this file.
No physical-machine uniqueness or authorship guarantee
is claimed by the accompanying evidence CSV.

Use synthetic student IDs only, such as `S1`. Do not enter real student records.

## Requirements

Full JDK 21, Git, an editor, and the separate Jenkins setup folder.
No Maven, Gradle, Docker, database, web framework, or cloud account is needed.
Before-class software setup is in `Student_Prerequisites_Standalone.pdf`.
The code is introduced during the lecture, not required for preparation.

## Build and run

Open a terminal inside `festpass-java`. Start with a freshly extracted project:

```text
java ci/Build.java verify
```

**Expect this to fail its tests initially.** Do not run `package` to bypass the
failure. Use Jenkins Test Result and the existing assertions to diagnose it.
After your fix passes all 28 tests, continue:

```text
java ci/SmokeTest.java
java -jar build/festpass.jar help
java -jar build/festpass.jar add-event java "Java Workshop" 2
java -jar build/festpass.jar add-event music "Campus Music" 1
java -jar build/festpass.jar list
java -jar build/festpass.jar book java S1
java -jar build/festpass.jar book java S2
java -jar build/festpass.jar book java S3
java -jar build/festpass.jar cancel java S1
java -jar build/festpass.jar book java S3
java -jar build/festpass.jar roster java
java -jar build/festpass.jar summary
```

The repaired app must pass **28 tests**. The first `book java S3` deliberately reports
SOLD OUT and exits with code 2; continue with cancellation, then retry.
Final summary: 2 events, 2 booked seats, 1 available seat. Running `list`
in a new terminal shows the same saved data.

The default file is `data/festpass.tsv`, relative to your working directory.
For an independent run, put `--data FILE` before the command:

```text
java -jar build/festpass.jar --data data/second-demo.tsv add-event java "Java Workshop" 2
```

Events cannot be added twice. If repeating the walkthrough, choose a new data
file or remove only your known demo data file after stopping all commands.
The `data` directory is ignored by Git; do not commit personal or runtime data.

## Commands and rules

| Command | Behavior |
|---|---|
| `add-event ID "TITLE" CAPACITY` | Unique event ID; capacity 1-10000; title 1-80 characters. |
| `list` | Show events, bookings, capacity, and available seats. |
| `book EVENT STUDENT` | One booking per student per event; reject duplicates and full events. |
| `cancel EVENT STUDENT` | Remove an existing booking and free its seat; reject missing bookings. |
| `roster EVENT` | Show students in registration order. |
| `summary` | Show event count and total booked/available seats. |

IDs are case-sensitive and use 1-32 ASCII letters, digits, underscores, or
hyphens. The same student may register for different events.
Exit codes: **0** success, **2** invalid input/business-rule rejection,
**3** storage failure. Errors go to stderr and do not claim success.

## Read the architecture

- `FestPass.java`: CLI parsing, output, exit codes, and load/mutate/save flow.
- `BookingService.java`: validation, event management, and booking rules.
- `BookingStore.java`: versioned UTF-8 TSV persistence and atomic file replacement.
- `FestPassTest.java`: unit, persistence, CLI, corruption, and lock tests.
- `ci/Build.java`: compile all Java sources with `-Xlint:all -Werror`, test, package.
- `ci/SmokeTest.java`: launch the real JAR seven times against temporary data.
- `ci/BuildEvidence.java`: records pseudonymous submission evidence for both outcomes.
- `Jenkinsfile`: Checkout -> Submission setup -> Test -> Build JAR -> Smoke test JAR -> Save artifact; always archive evidence afterward.

Persistence format: `FESTPASS<TAB>1` header; `EVENT<TAB>ID<TAB>TITLE<TAB>CAPACITY`
records followed by `BOOKING<TAB>EVENT_ID<TAB>STUDENT_ID` records.
Titles cannot contain tabs/newlines. Bad records, unknown versions, duplicate
bookings, and over-capacity files are rejected instead of silently reset.

The CLI holds a sidecar file lock throughout each load/change/save operation.
It writes a temporary file in the same directory and atomically replaces the
store; unsupported atomic moves fail explicitly. Use a local disk, not a network
share or synchronized folder. Lock files may remain on disk; the OS lock, not
the file's presence, determines whether a command is active. Always use the same
data path, not alternate symlink aliases.

This remains a teaching CLI: no authentication, distributed database, web UI,
payment system, or production reservation guarantee is claimed.

## Make the first commit and connect Jenkins

```text
git init -b main
git add .
git commit -m "Add FestPass starter and pipeline"
```

If Git asks for identity, set your own repository-local `user.name` and
`user.email`, then retry. No GitHub account or push is needed.

Follow `jenkins-lab/README.md` for the local Git server, Jenkins controller,
separate agent, and Pipeline-from-SCM job. Add and commit the polling trigger
as described in **ASSIGNMENT.md** and guide section 7. Then run **Build Now once**
to load that Jenkinsfile and establish polling. Without your trigger, the
starter job does not automatically build new commits. Later local commits should trigger builds
within approximately 1-2 minutes. Saving a file alone is not enough.

## Diagnose the seeded failure, then recover

The initial source already contains a defect. Do not introduce another bug.
Capture an automatic failing run after your harmless help-text change.
Use failed assertions to locate and fix the faulty business rule. Commit
your repair, then observe an automatic successful run with all tests passing.

The failed run must not publish a new JAR, but must publish
`submission-evidence.csv`. Older successful JARs remain older artifacts.
Do not delete tests or weaken the pipeline to make it green.

The tests use a dependency-free runner that writes JUnit-compatible XML, not
the JUnit test framework. New test files are compiled automatically, but new
test cases must be registered in the `FestPassTest` runner.

`java ci/Build.java test` compiles and tests without packaging.
`java ci/Build.java package` packages without tests; Jenkins runs it only after
the Test stage succeeds. `verify` tests and packages locally. Run the separate
smoke command after `verify` to validate the actual JAR.

## My submission

Author: Tanishq Choudhary. Job: `festpass-ci` (Pipeline script from SCM,
`git://127.0.0.1:9418/festpass-java`, branch `*/main`, agent `student-laptop`).

### What I did

1. Set up Jenkins 2.580.1, the Git server and the `student-laptop` agent on my laptop.
2. Filled in `submission.properties` and added `triggers { pollSCM('* * * * *') }` to the Jenkinsfile.
3. Ran Build Now once to load the trigger. It failed at the Test stage, as expected.
4. Changed the help intro line to include my name and committed. Polling started the next build, which failed the same way.
5. Read the failing tests, found the bug, fixed it, committed. Polling started a green build.

| Build | Started by | Commit | Result |
|---|---|---|---|
| 1 | user, once | dd3304c | FAILURE, 3 of 28 tests failed |
| 2 | SCM change | a37b54c | FAILURE, same 3 tests |
| 3 | SCM change | 7a7f3e1 | SUCCESS, 28 passed, JAR built |

### The bug

`cancel()` in `BookingService.java` checked that the booking existed but never removed
the student, so the seat stayed taken and the cancellation was lost after reload.
The fix is one line at the end of `cancel()`: `event.students.remove(studentId);`.
Tests, pipeline stages and other files were not changed, apart from the polling trigger.

### Where the evidence is

The `submission` folder holds `EVIDENCE.md`, the evidence CSV for each build
(`build1-red`, `build2-red`, `build3-green`) and the final `festpass.jar`.

### How to check the codes and the evidence

- The two codes are in `submission.properties` as `class.code` and `challenge.code`.
  `ci/BuildEvidence.java` accepts 3 to 64 letters, digits, `_` or `-` and rejects the
  `_REQUIRED` placeholders, so a build with a bad code fails at Submission setup.
- Each evidence CSV repeats them in the `ClassCode` and `ChallengeCode` columns.
- The `StudentAlias` column comes from the commit author. Check it with `git log -1 --format=%an`.
- The `Commit` column should match the commit Jenkins built. For build 3 that is
  `7a7f3e1097d6bb9c096017de8f9477aaa479fb11`; compare with `git log --format=%H`.
- The `JarSha256` column in `build3-green/submission-evidence.csv` should equal the hash of the JAR:

```text
Get-FileHash submission/build3-green/festpass.jar -Algorithm SHA256
```

- Run the JAR to see the personalized help line:

```text
java -jar submission/build3-green/festpass.jar help
```

To rerun everything locally: `java ci/Build.java verify`, then `java ci/SmokeTest.java`.
