# Connector Development Kit \(Python\)

The Airbyte Python CDK is a framework for rapidly developing production-grade Airbyte connectors. The CDK currently offers helpers specific for creating Airbyte source connectors for:

* HTTP APIs \(REST APIs, GraphQL, etc..\)
* Singer Taps
* Generic Python sources \(anything not covered by the above\)

The CDK provides an improved developer experience by providing basic implementation structure and abstracting away low-level glue boilerplate.

This document is a general introduction to the CDK. Readers should have basic familiarity with the [Airbyte Specification](https://docs.airbyte.com/understanding-airbyte/airbyte-protocol/) before proceeding.

## Getting Started

Generate an empty connector using the code generator. First clone the Airbyte repository then from the repository root run

```text
cd airbyte-integrations/connector-templates/generator
./generate.sh
```

then follow the interactive prompt. Next, find all `TODO`s in the generated project directory -- they're accompanied by lots of comments explaining what you'll need to do in order to implement your connector. Upon completing all TODOs properly, you should have a functioning connector.

Additionally, you can follow [this tutorial](https://docs.airbyte.io/connector-development/tutorials/cdk-tutorial-python-http) for a complete walkthrough of creating an HTTP connector using the Airbyte CDK.

### Concepts & Documentation

See the [concepts docs](docs/concepts/) for a tour through what the API offers.

### Example Connectors

**HTTP Connectors**:

* [Exchangerates API](https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-exchange-rates/source_exchange_rates/source.py)
* [Stripe](https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-stripe/source_stripe/source.py)
* [Slack](https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-slack/source_slack/source.py)

**Singer connectors**:

* [Salesforce](https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-salesforce-singer/source_salesforce_singer/source.py)
* [Github](https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-github-singer/source_github_singer/source.py)

**Simple Python connectors using the barebones `Source` abstraction**:

* [Google Sheets](https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-google-sheets/google_sheets_source/google_sheets_source.py)
* [Mailchimp](https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-mailchimp/source_mailchimp/source.py)

## Contributing

### First time setup

We assume `python` points to python &gt;=3.8.

Setup a virtual env:

```text
python -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]" # [dev] installs development-only dependencies
```

#### Iteration

* Iterate on the code locally
* Run tests via `python -m pytest -s unit_tests`
* Perform static type checks using `mypy airbyte_cdk`. `MyPy` configuration is in `.mypy.ini`.
* The `type_check_and_test.sh` script bundles both type checking and testing in one convenient command. Feel free to use it!

##### Autogenerated files
If the iteration you are working on includes changes to the models, you might want to regenerate them. In order to do that, you can run:
```commandline
SUB_BUILD=CDK ./gradlew format
```
This will generate the files based on the schemas, add the license information and format the code. If you want to only do the former and rely on
pre-commit to the others, you can run the appropriate generation command i.e. `./gradlew generateComponentManifestClassFiles`.

#### Testing

All tests are located in the `unit_tests` directory. Run `python -m pytest --cov=airbyte_cdk unit_tests/` to run them. This also presents a test coverage report.

#### Building and testing a connector with your local CDK

When developing a new feature in the CDK, you may find it helpful to run a connector that uses that new feature. You can test this in one of two ways:
* Running a connector locally
* Building and running a source via Docker

##### Installing your local CDK into a local Python connector

In order to get a local Python connector running your local CDK, do the following.

First, make sure you have your connector's virtual environment active:
```bash
# from the `airbyte/airbyte-integrations/connectors/<connector-directory>` directory
source .venv/bin/activate

# if you haven't installed dependencies for your connector already
pip install -e .
```

Then, navigate to the CDK and install it in editable mode:
```bash
cd ../../../airbyte-cdk/python
pip install -e .
```

You should see that `pip` has uninstalled the version of `airbyte-cdk` defined by your connector's `setup.py` and installed your local CDK. Any changes you make will be immediately reflected in your editor, so long as your editor's interpreter is set to your connector's virtual environment.

##### Building a Python connector in Docker with your local CDK installed

You can build your connector image with the local CDK using
```bash
# from the airbytehq/airbyte base directory
CONNECTOR_TAG=<TAG_NAME> CONNECTOR_NAME=<CONNECTOR_NAME> sh airbyte-integrations/scripts/build-connector-image-with-local-cdk.sh
```
Note that the local CDK is injected at build time, so if you make changes, you will have to run the build command again to see them reflected.

##### Running Connector Acceptance Tests for a single connector in Docker with your local CDK installed

To run acceptance tests for a single connectors using the local CDK, from the connector directory, run
```bash
LOCAL_CDK=1 sh acceptance-test-docker.sh
```
To additionally fetch secrets required by CATs, set the `FETCH_SECRETS` environment variable. This requires you to have a Google Service Account, and the GCP_GSM_CREDENTIALS environment variable to be set, per the instructions [here](https://github.com/airbytehq/airbyte/tree/b03653a24ef16be641333380f3a4d178271df0ee/tools/ci_credentials).

##### Running Connector Acceptance Tests for multiple connectors in Docker with your local CDK installed

To run acceptance tests for multiple connectors using the local CDK, from the root of the `airbyte` repo, run
```bash
./airbyte-cdk/python/bin/run-cats-with-local-cdk.sh -c <connector1>,<connector2>,...
```

#### Publishing a new version to PyPi

1. Open a PR
2. Once it is approved and **merged**, an Airbyte member must run the `Publish CDK Manually` workflow from master using `release-type=major|manor|patch` and setting the changelog message.

## Coming Soon

* Full OAuth 2.0 support \(including refresh token issuing flow via UI or CLI\) 
* Airbyte Java HTTP CDK
* CDK for Async HTTP endpoints \(request-poll-wait style endpoints\)
* CDK for other protocols
* Don't see a feature you need? [Create an issue and let us know how we can help!](https://github.com/airbytehq/airbyte/issues/new?assignees=&labels=type%2Fenhancement&template=feature-request.md&title=)