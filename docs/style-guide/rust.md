# Rust

## Code style

Mainly you just need to follow [standard Rust styling and formatting rules](https://github.com/rust-lang/rust/tree/master/src/doc/style-guide/src) and make your code easy to understand. It is recommended that you enable auto format on save in your editor.

Follow [clippy](https://doc.rust-lang.org/stable/clippy/usage.html) lints (this is enforced by CI). Note: sometimes new lints are added in Rust updates, you don't need to fix them all right away in unrelated code as long as the CI succeeds.

### Error handling

User-facing errors should be handled with Result with the Issue or ErrorResponse types. Those are standard FHIR errors that will display in the correct format to the user. Using unwrap is generally discouraged, but is allowed if there is a good reason on why it should never fail. If you do want to use it, you should use `expect` with a message on what went wrong. Avoid ignoring errors (all errors should at least be written to logs).

### Services

Handlers (REST or GRPC) should be as thin as possible, all business logic should be in a separate module.

### Security

We should follow OWASP principles as described at [https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/](https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/)


### Test coverage

Each part of business logic in services should be covered by unit or functional test.
Complex features, which require several services interaction should be covered by [behavioural tests](git@github.com:edenlabllc/kodjin.tests.git)

Test coverage in services can be calculated with [tarpaulin](https://github.com/xd009642/tarpaulin) tool

For libraries acceptable test coverage is above 60%. For web services and non web services there is no strict requirement for test coverage.
