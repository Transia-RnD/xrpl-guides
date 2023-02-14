# Xrpl Testing Guide

## Building the testfile

Depending on the changes you make you will likely need to create a testfile. Before making any files, ask yourself;

- Does this proposal add any ledger objects methods? (testfile)
- Does this proposal add any rpc methods? (testfile)

### Create a testfile.cpp

1. Create a new file. Name that file `[FEATURE]_test.cpp`.

Paste the following in the testfile and rename [FEATURE] to your feature name.

```
//------------------------------------------------------------------------------
/*
    This file is part of rippled: https://github.com/ripple/rippled
    Copyright (c) 2012, 2013 Ripple Labs Inc.

    Permission to use, copy, modify, and/or distribute this software for any
    purpose  with  or without fee is hereby granted, provided that the above
    copyright notice and this permission notice appear in all copies.

    THE  SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
    WITH  REGARD  TO  THIS  SOFTWARE  INCLUDING  ALL  IMPLIED  WARRANTIES  OF
    MERCHANTABILITY  AND  FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
    ANY  SPECIAL ,  DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
    WHATSOEVER  RESULTING  FROM  LOSS  OF USE, DATA OR PROFITS, WHETHER IN AN
    ACTION  OF  CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
    OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
*/
//==============================================================================

#include <ripple/basics/chrono.h>
#include <ripple/ledger/Directory.h>
#include <ripple/protocol/Feature.h>
#include <ripple/protocol/Indexes.h>
#include <ripple/protocol/TxFlags.h>
#include <ripple/protocol/jss.h>
#include <test/jtx.h>

#include <chrono>

namespace ripple {
namespace test {
struct [Feature]_test : public beast::unit_test::suite
{
    // ...get keylets (ledger entries or objects)

    // ...get ownerdir (owner objects)

    // ... methods on feature

    void
    testEnabled(FeatureBitset features)
    {
        testcase("enabled");
        using namespace jtx;
        // ... your disabled/enabled tests will go here
        // to learn about setting up the test env visit.
    }

    void
    testWithFeats(FeatureBitset features)
    {
        testEnabled(features);
    }

public:
    void
    run() override
    {
        using namespace test::jtx;
        auto const sa = supported_amendments();
        testWithFeats(sa);
    }
};

BEAST_DEFINE_TESTSUITE(URIToken, app, ripple);
}  // namespace test
}  // namespace ripple

```

2. Add the file to the cmake build file. (conan may not require this)

The files need to be in alphabetical order.

```
A_test.cpp
`[FEATURE]_test.cpp`
B_test.cpp
```

3. Add the structure/class.

All tests need to be run with features env. You can find the tests that will run in the `run` and `testWithFeats` setup functions.

Then you will structure the test file with the following test functions.

// <Proposal Basic Tests>
`testEnabled`: Test the amendment is enabled. If using a token test both here.
`testInvalids`: Test errors, Func1Invalids, Func2Invalids
`testMetadataOwnership`: Test MetaData & Ownership
// <Proposal Features Tests>
`testFlags`: Test the specific flags enabled with the amendment.
`testMethods`: Test the specific methods enabled with the amendment. Func1, Func2
`testMethodsOptionals`: Test the specific methods enabled with the amendment with optionals. Func1, Func2
// <Native Token Tests>
`testDisallowXRP?`: Test Disallow XRP flag on dest
// <Fungible Token Tests>
`testDepositAuth?`: Test Deposit Auth
`testTransferFee?`: Test TransferFee
`testRequireAuth?`: Test RequireAuth
`testFreeze?`: Test Freeze (Global & Trustline)
`testDefaultRipple?`: Test Ripple (Default)
`testLimitAmount?`: Test LimitAmount
`testGateway?`: Test Issuer -> Alice
`testPrecision?`: Test Precision Loss on transfer
`testRippleState?`: Test State of Ripple after an xfer
// <Global Misc Tests>
`testDstTag?`: Test Destination Tag
`testDeleteAccount?`: Test Deleting the Account
`testTickets?`: Test Tickets

If you need to update or add rpc methods you need to add the test to the respective `rpc` testfile. * You may need to create a file and add it to the cmake build file.

// <RPC Method Tests>
`testFeatureRPC?`: Test the rpc changes made by the proposal. This could be `AccountObjects` or a new rpc created.
`testFeatureRPCMarkers?`: Test the rpc markers on the objects created/changed by the proposal

> Some of the functions above are placeholders. Something like `testMethods` does not mean test all the methods in one test but in seperate tests.

`testFunc1`
`testFunc2`

## Creating Setup Test Runner

### ENV w/ Features

`Env env(*this, features);`

### Accounts

```
auto const alice = Account("alice");
auto const bob = Account("bob");
auto const gw = Account("gateway");
```

### Currencies

```
auto const USD = gw["USD"];
auto const EUR = gw["EUR"];
```


### Native Fund

```
env.fund(XRP(10000), alice, bob);
env.close();
```

### Token Fund

```
env.fund(XRP(10000), alice, bob, gw);
env.close();
env.trust(USD(100000), alice);
env.trust(USD(100000), bob);
env.close();
env(pay(gw, alice, USD(10000)));
env(pay(gw, bob, USD(10000)));
env.close();
```

### Full IC ENV Example.

```
Env env(*this, features);
auto const alice = Account("alice");
auto const bob = Account("bob");
auto const gw = Account("gateway");
auto const USD = gw["USD"];
env.fund(XRP(10000), alice, bob, gw);
env.close();
env.trust(USD(100000), alice);
env.trust(USD(100000), bob);
env.close();
env(pay(gw, alice, USD(10000)));
env(pay(gw, bob, USD(10000)));
env.close();
```

### Setting/Clearing Account Flags 

Set Account Flags

`env(fset(gw, asfGlobalFreeze));`

Clear Account Flags

`env(fclear(gwF, asfGlobalFreeze));`

### Adding Flags to transactions

`env(trust(gw, bobUSD(10000)), txflags(tfSetfAuth));`

### Checking Balances

Balance against XRP or Token

`env.require(balance(alice, USD(4000)));`

Balance with variables

`env.require(balance(alice, XRP(4000) - drops(10)));`

`env.require(balance(alice, USD(4000) - USD(10)));`

Pre & Post for transaction result variables

```
auto const preValue = myFunction(env, alice, gw, USD);
// env function call ex. env(pay(...));
// env close ex. env.close();
auto const postValue = myFunction(env, alice, gw, USD);
env.require(postValue, preValue + USD(1000));
```

### Creating Methods

Example below is creating a test transaction to submit using `env(func(var, var))`.

```
static Json::Value
func(
    jtx::Account const& account,
    std::string const& uri)
{
    using namespace jtx;
    Json::Value jv;
    jv[jss::TransactionType] = jss::URIToken;
    jv[jss::Flags] = tfBurnable;
    jv[jss::Account] = account.human();
    jv[sfURI.jsonName] = strHex(uri);
    return jv;
}
```

### Checking Owner Directory

`inOwnerDir`: tells us if the object is in the owners directory (Account).
`ownerDirCount1`: tells us the number of objects in the users directory

```
static bool
inOwnerDir(ReadView const& view, jtx::Account const& acct, std::shared_ptr<SLE const> const& token)
{
    ripple::Dir const ownerDir(view, keylet::ownerDir(acct.id()));
    return std::find(ownerDir.begin(), ownerDir.end(), token) != ownerDir.end();
}
static std::size_t
ownerDirCount(ReadView const& view, jtx::Account const& acct)
{
    ripple::Dir const ownerDir(view, keylet::ownerDir(acct.id()));
    return std::distance(ownerDir.begin(), ownerDir.end());
};
```

### Checking Ledger Entries

Example below is reading the PayChan ledger entry at `chan` which is the hash/id.

```
static STAmount
channelBalance(ReadView const& view, uint256 const& chan)
{
    auto const slep = view.read({ltPAYCHAN, chan});
    if (!slep)
        return XRPAmount{-1};
    return (*slep)[sfBalance];
}
```

### Checking Keylets

Example below is reading the users trustline and getting the value for `sfLockedBalance`.

```
static STAmount
lockedAmount(
    jtx::Env const& env,
    jtx::Account const& account,
    jtx::Account const& gw,
    jtx::IOU const& iou)
{
    auto const sle = env.le(keylet::line(account, gw, iou.currency));
    return -(*sle)[sfLockedBalance];
}
```

### Testing Tx Submit Failures

Add the `ter` argument after the function call.

`env(claim(bob, chan, delta, delta), ter(temBAD_SIGNATURE));`

Drain the account of IC & TL

```
env(pay(gw, alice, env.balance(alice, USD.issue())));
env.trust(USD(0), alice);
```

Ripple State

Balance is;

Negative: Low Account is issuer
Positive: High Account is issuer