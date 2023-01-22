# XRPL - RIPPLED Cheatsheet

## Setup ENV

`xcode-select --install`

`brew update`

`brew install git cmake pkg-config protobuf openssl ninja`

```
cd /LOCATION/OF/YOUR/BOOST/DIRECTORY
./bootstrap.sh
./b2 cxxflags="-std=c++14"
```

`sudo nano ~/.zshrc`

```
echo "export BOOST_ROOT=/Users/dustedfloor/projects/BoostBuilds/boost_1_77_0" >> ~/.zshrc
```

`source ~/.zshrc`

Create Build and Config
`mkdir build && cd build && cmake -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=Debug ..`

Build and test single file
`cmake --build . -- -j 4 && ./rippled -u ripple.app.PayChan`

Run one test

`./rippled --unittest=ripple.app.PayChan`
`./rippled --unittest=ripple.app.Escrow`
`/rippled --unittest=ripple.tx.NFToken`
`/rippled --unittest=ripple.app.RPCCall`

## Debug in M1

Load up the rippled build

`lldb ./rippled`

Run the unit test file

`run -u ripple.app.Escrow`

Set Breakpoint to file

`breakpoint set --file tx/impl/PayChan.cpp -l 294`
`breakpoint set --file tx/impl/Escrow.cpp -l 643`
`breakpoint set --file ledger/View.h -l 887`
`breakpoint set --file net/impl/RPCCall.cpp -l 799`
`breakpoint set --file rpc/handlers/PayChanClaim.cpp -l 79`
`breakpoint set --file rpc/handlers/AccountChannels.cpp -l 79`

Step in BP

`continue`

Print Out

`std::cout << "PRE ALICE: "" << preAlice << "\n";`

## Writing Tests


### Test Runner - Funding & Setup

Native Fund

```
Env env(*this, features);
auto const alice = Account("alice");
auto const bob = Account("bob");
env.fund(XRP(10000), alice, bob);
env.close();
```

Token Fund

```
auto const alice = Account("alice");
auto const bob = Account("bob");
auto const gw = Account{"gateway"};
auto const USD = gw["USD"];
Env env(*this, features);
env.fund(XRP(10000), alice, bob, gw);
env.close();
env.trust(USD(100000), alice);
env.trust(USD(100000), bob);
env.close();
env(pay(gw, alice, USD(10000)));
env(pay(gw, bob, USD(10000)));
env.close();
```

Set Account Flags

`env(fset(gw, asfGlobalFreeze));`

Clear Account Flags

`env(fclear(gwF, asfGlobalFreeze));`

Add Flags to tx

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