# WORKING WITH SDKS

## JS

### Run independent test

Update the package.json integration script to the the following to test a single file:

`"test:integration": "TS_NODE_PROJECT=tsconfig.build.json nyc mocha ./test/integration/fundWallet.test.ts",`

## Python

### Run independent test

poetry run poe test_integration

```
source .venv/bin/activate && \
python3 -m unittest tests/integration/reqs/test_channel_request.py
```

python3 -m unittest tests/integration/reqs/test_channel_verify.py
python3 -m unittest tests/integration/reqs/test_channel_request.py

```
source .venv/bin/activate && \
python3 -m unittest tests/unit/core/binarycodec/test_main.py
```