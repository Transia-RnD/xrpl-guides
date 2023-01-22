# WORKING WITH SDKS

## JS

### Run independent test

Update the package.json integration script to the the following to test a single file:

`"test:integration": "TS_NODE_PROJECT=tsconfig.build.json nyc mocha ./test/integration/fundWallet.test.ts",`

## Python

### Run independent test

```
source .venv/bin/activate && \
python3 -m unittest tests/integration/transactions/test_payment_channel_create.py
```
```
source .venv/bin/activate && \
python3 -m unittest tests/unit/core/binarycodec/test_main.py
```