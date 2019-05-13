When making changes to the code, please run the appropriate tests, described below, before submitting a PR.

**Note:** We run all tests in release mode (indicated by the `--release` flag). See [Building The Libraries](#building-the-libraries) for more information.

## Mock routing

If a change is minor or does not involve potential breakage in data or API compatibility, it is not necessary to test against the actual network. In this case, it is enough to run unit tests using mock routing and make sure that they all pass. To do this, you will have to go to the crate you wish to test (e.g. the `safe_core` directory) and execute the following:

```shell
cargo test --release --features=mock-network
```

This will run all unit tests for the crate.

## Real network

When in doubt, perform an integration test against the real network, in addition to mock routing tests above. This will test various network operations, so please make sure you have some available account mutations. The steps:

- You will need a SAFE network account. You can register one from within the [SAFE Browser](https://github.com/maidsafe/safe_browser/releases).
- Make sure you are able to login to your account via the SAFE Browser.
- Navigate to the `tests` directory.
- Open `tests.config` and set your account locator and password. **Do not commit this.** Alternatively, you can set the environment variables `TEST_ACC_LOCATOR` and `TEST_ACC_PASSWORD`.
- Run `cargo test --release authorisation_and_revocation -- --ignored --nocapture` and make sure that no errors are reported.

### Binary compatibility of data

You may need to test whether your changes affected the binary compatibility of data on the network. This is necessary when updating compression or serialization dependencies to make sure that existing data can still be read using the new versions of the libraries.

- Set your account locator and password as per the instructions above.
- Run the following command on the **master** branch: `cargo test --release write_data -- --ignored --nocapture`
- On the branch with your changes, ensure the following command completes successfully: `cargo test --release read_data -- --ignored --nocapture`

## Viewing debug logs

The codebase contains instrumentation statements that log potentially useful debugging information, at different priorities. Such statements include the macros `debug!` and `trace!`, and more can be found at the documentation for the [log crate](https://docs.rs/log). Feel free to add trace calls to the code when debugging.

In order to view the output of trace calls you will need to initialize logging at the beginning of your test:

```rust
unwrap!(maidsafe_utilities::log::init(true));
```

Then you will need to set the `RUST_LOG` environment variable to the desired logging level for the desired modules or crates. To view trace calls for `safe_authenticator`, for example, you may do this:

```shell
export RUST_LOG=safe_authenticator=trace
```

You could also set `RUST_LOG=trace` which will output *all* trace logs, but this may produce far more data than desired. For more information please see the documentation for the `log` module in our crate [maidsafe_utilities](https://docs.rs/maidsafe_utilities).
