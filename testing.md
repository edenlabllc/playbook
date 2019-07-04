### Regression test goals
1. Improve delivery quality
2. Decrease release/test turnaround cycle
3. Decrease regression testing cost
4. Make regression tests person-independent

### Key points
1. Test scenarios should go along with the use-cases/specifications as an input for the development task.
2. Test scenarios should be prepared by SA
3. Step definitions should be implemented by Dev.
4. Different tool and approach might be used for the UI testing
5. We don’t want to duplicate test scenarios for end-2end tests and for integration tests (if it is about the same).
6. Test scenarios should be judged the same way as application source code. It should be stored in the same repository with the source code. We’ll start with:
    1. ehealth.api
    2. ehealth.web
    3. medical_events
7. Integration tests should be environment agnostic. It should be possible to run it on any existing environment (dev/devmo/preprod/local).
8. We have a number of scenarios already implemented for the end-2-end tests on elixir. If we manage to reuse it, we should move with the step definition on Elixir.
    1. If not -python/golang
9. Fixtures
    1. We have no working solution that looks ok for all the team members. That’s why we’ll start with one of the options
        1. A set on insert scripts that should be performed as a prerequisites to run tests
        2. Run tests on the existing DB copy.
            1. Using selects to get the required data
            2. Manually fill-in the configuration
10. Test suit should be part of CD. It should generate human-readable report after deployment (configurable - for eample, we probably won't do it on dev).
