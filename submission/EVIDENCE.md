# Jenkins assignment evidence

Name (Git author): Tanishq Choudhary
Class code: 001
Repository: https://github.com/Tanishq-Choudhary/festpass-java
Job: festpass-ci, Pipeline script from SCM, git://127.0.0.1:9418/festpass-java, branch */main, script path Jenkinsfile, agent student-laptop

## Builds

| Build | Started by | Commit | Result | Tests |
|---|---|---|---|---|
| 1 | user (Build Now, once, to load the trigger) | dd3304c Configure my Jenkins assignment | FAILURE | 28 run, 3 failed |
| 2 | SCM change (polling) | a37b54c Personalize my help introduction | FAILURE | 28 run, 3 failed |
| 3 | SCM change (polling) | 7a7f3e1 Fix cancel so it removes the booking and frees the seat | SUCCESS | 28 run, 0 failed |

The commit Jenkins checked out matched `git log` each time. I did not press Build Now for builds 2 and 3.

## Failing assertion (builds 1 and 2)

`cancellationFreesSeat` failed with: "SOLD OUT. Even the developer needs a ticket."
The other two failures were `cancellationsSurviveReload` ("Cancellation was lost") and `cliWorkflowPersistsAcrossCommands` ("List does not reflect persisted cancellation").

## Skipped stages in the failed builds

Build JAR, Smoke test JAR and Save artifact were skipped, so no JAR was published. The evidence CSV was still archived.

## What was wrong and what I changed

The rule is that a cancelled booking gives the seat back. `cancel()` in `BookingService.java` checked that the student had a booking, but it never removed the student from the event. So the seat stayed taken, S2 got "SOLD OUT" after S1 cancelled, and the cancellation was gone after saving and reloading.

I added one line at the end of `cancel()`: `event.students.remove(studentId);`. I changed no tests, no pipeline stages and no other files. The only change to the Jenkinsfile is the pollSCM trigger.

Before committing I ran `java ci/Build.java verify` (28 tests, 0 failures) and `java ci/SmokeTest.java` (passed).

## Final JAR

`java -jar festpass.jar help` prints:

```
FestPass - campus event registration (Tanishq Choudhary)
```

## Files

- build1-red/submission-evidence.csv
- build2-red/submission-evidence.csv
- build3-green/submission-evidence.csv
- build3-green/festpass.jar
- Repository: festpass-java (commits dd3304c, a37b54c, 7a7f3e1, plus the starter commit ab7f2b2)
