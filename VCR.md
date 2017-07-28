# How to Use VCR


## Overview

VCR is an integration testing tool that will record HTTP interactions
with external services in re-playable "cassettes." Tests can be
executed against playback of the cassettes in situations where it's not
convenient to integrate with the external service, either for
performance or isolation (e.g. on a CI server).

### VCR is NOT a mocking tool.

Ok, it sorta is, but if you're _only_ using for mocking, it's a
relatively expensive, brittle way to go.

If you just need to mock or stub out an external service altogether,
use a mocking library like mocha. Don't use VCR.

If you can afford to run your integration tests against an actual
instance of the external service all the time, don't use VCR.

Tests that use VCR should be able to be re-run against the actual
external service at any time.

## Tests Need To Be Self-Sufficient

No tests, unit or otherwise, should depend on magic external data,
whether that's in its own database, or in an external service. Each
test should setup and teardown the data it depends on.

If tests are relying on magic data recorded in cassettes, then those
cassettes cannot be easily re-recorded, and that codebase has lost the
ability to quickly verify that no contracts between the two services
have been broken.

## Automatic Re-Recording

See this section of the VCR docs for more information about how to
configure VCR to re-record cassettes at regular intervals.

## FAQ

### _Above it says VCR is an expensive mocking tool. How so?_

The cassettes are recording a lot of data. Maintaining the cassettes
over the lifetime of an app can be a pain. I've had problems where .yml
cassettes didn't maintain compatibility through versions of Ruby or
other dependencies, and files needed to be re-processed, due to
character encoding problems. I've had problems where switching to JSON
was better to support encoding issues.

If the expected external behavior changes, working with mocha is
typically simple to do, in comparison with re-recording a cassette or
trying to manually edit it.

Debugging through a lot of recorded HTTP behavior can be more
time-consuming than a proper mocking library.

In the small, over a few days session in a codebase, working with VCR
may not cost anything additional. Usually the issues I run into are
problems over the long haul.

### _So, why use VCR at all?_

Actual integration testing is a good thing. Overuse of mocks can lead
to a test suite that fools one into thinking everything is fine with
changes made, without doing anything to actually verify integrations
with external services. If you have a reliable test or staging instance
to always run the integration tests against, then perhaps you don't
need to use VCR. Those tests may perform slower, but if the trade-off
is a gain in faster feedback when the staging instance gets a deployed
version that breaks integration, that could be worth it.

If you don't have reliable test instances of external services (either
in dev or CI), or the performance is a beating, then using VCR to
playback data from the external service is a good fit, but be sure to
establish a process to re-record the cassettes at regular intervals.

### _How do I use Automatic Re-Recording in a CI environment?_

You may not be able to. If VCR is buying you isolation from the
external service because the CI box has no access to a test or staging
instance of the app, then you'll have to re-record the cassettes in a
dev environment.
