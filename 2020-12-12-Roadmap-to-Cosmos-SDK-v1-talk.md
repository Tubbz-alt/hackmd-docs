---
slideOptions:
  theme: solarized
  transition: 'fade'
  # parallaxBackgroundImage: 'https://s3.amazonaws.com/hakim-static/reveal-js/reveal-parallax-1.jpg'
---

# The Road to Cosmos SDK v1.0
## Aaron Craelius
### CTO, Regen Network
### Core Maintainer of the Cosmos SDK

----

# Regen Netowrk

- Blockchain platform for issuing and managing ecosystem serivce credits, and 
- Since April 2020, Regen has been acting as the lead maintainer of the Cosmos SDK

---


# What does 1.0 mean?

---

## 1.0 is a commitment to stability

----

## Maybe it's obvious, but why is stability important?
We can't build durable, long-lived systems without stability.
We need a solid foundation to start building other layers.

---

## 1.0 is a commitment to a great developer experience

----

Blockchain is hard! A successful blockchain development platform __must__ have a strong focus on DevX to balance against the inherant complexity of developing distributed ledger technologies.

---


## Stability & DevX for who?

* Client Developers
  * developers building user-facing apps that integrate with Cosmos SDK blockchains
* Module Developers
  * developers building custom modules for Cosmos SDK blockchains
* App Developers
  * developers building Cosmos SDK blockchains

---


## How Stargate moved us closer to 1.0:
* **Protobuf**: a standard multi-platform protocol for clients
* **Live upgrades**: blockchains can gracefully upgrade without needing to start over from genesis

---

## Stargate for Module Developers
**Simplified workflow for implementing a module:**
1. Write proto definitions
2. Generate code
3. Implement interfaces in generated code
4. Wire up services in single `RegisterServices` entry point

----

## 1. Write proto definitions

```proto
package cosmos.bank;

// Msg defines the bank Msg service.
service Msg {
  // Send defines a method for sending coins from one account to another account.
  rpc Send(MsgSend) returns (MsgSendResponse);

  // MultiSend defines a method for sending coins from some accounts to other accounts.
  rpc MultiSend(MsgMultiSend) returns (MsgMultiSendResponse);
}

message MsgSend {
  string   from_address                    = 1;
  string   to_address                      = 2;
  repeated cosmos.base.v1beta1.Coin amount = 3;
}

message MsgSendResponse {}

message MsgMultiSend {
  repeated Input  inputs  = 1;
  repeated Output outputs = 2;
}

message MsgMultiSendResponse {}
```

----

## 2. Generate code

```go=
package bank;

// MsgServer is the server API for Msg service.
type MsgServer interface {
	// Send defines a method for sending coins from one account to another account.
	Send(context.Context, *MsgSend) (*MsgSendResponse, error)
	// MultiSend defines a method for sending coins from some accounts to other accounts.
	MultiSend(context.Context, *MsgMultiSend) (*MsgMultiSendResponse, error)
}
```

----

## 3. Implement interfaces in generated code

```go=
func (k msgServer) Send(ctx context.Context, msg *types.MsgSend) (*types.MsgSendResponse, error) {
	...
}

func (k msgServer) MultiSend(goCtx context.Context, msg *types.MsgMultiSend) (*types.MsgMultiSendResponse, error) {
	...
}

```

----

## 4. Wire up services in single `RegisterServices` entry point

```go=
func (am AppModule) RegisterServices(cfg module.Configurator) {
	types.RegisterMsgServer(cfg.MsgServer(), keeper.NewMsgServerImpl(am.keeper))
	types.RegisterQueryServer(cfg.QueryServer(), am.keeper)
}
```

---

## Stargate for Client Developers

* easy client libraries in all languages
* backwards compatible APIs: breaking change detection is run against every commit to prevent breaking changes to `v1` proto files
* easy to implement transaction signing (`SIGN_MODE_DIRECT`)
* gRPC + auto-generated REST/OpenAPI support

----

## Typescript generated code

```typescript
/**
 *  Msg defines the bank Msg service.
 */
export interface Msg {

  /**
   *  Send defines a method for sending coins from one account to another account.
   */
  Send(request: MsgSend): Promise<MsgSendResponse>;

  /**
   *  MultiSend defines a method for sending coins from some accounts to other accounts.
   */
  MultiSend(request: MsgMultiSend): Promise<MsgMultiSendResponse>;

}
```

----

## Automatic REST/OpenAPI Endpoints

![](https://i.imgur.com/uPCvGrY.png)

---

## Stargate for App Developers

**`app.go` is pretty massive 🤔**

![](https://i.imgur.com/2p1QHT3.png)

---

## Stargate was about setting a foundation
## But there's a lot work to do...

---

### 1.0 for Client Devs: Protobuf
* stabilizing proto files as `v1` (they're currently mostly `v1beta1`)
* proto files for individual modules will likely reach `v1` before the SDK and when they do clients can expect stability

---

### 1.0 for Client Devs: Transactions
* need to design `SIGN_MODE_TEXTUAL` to fully replace Amino JSON
* improve multisig signing UX
* standardized public key address format for new signature algorithms

---

### v1.0 for Module Developers

* protobuf code-generation improvements - better handling of `Any`s and custom types
* simplify and stabilize core `types/` package
    * remove things that don't belong there
    * better decimal support
* simplify and improve the `AppModule` interface and inter-module wiring

----

### Inter-module communication
Protobuf generates client golang interfaces:

```go=
type MsgClient interface {
	Send(ctx context.Context, in *MsgSend, opts ...grpc.CallOption) (*MsgSendResponse, error)
	MultiSend(ctx context.Context, in *MsgMultiSend, opts ...grpc.CallOption) (*MsgMultiSendResponse, error)
}
```

* Modules could talk to other modules using these interfaces
* This could greatly simplify `app.go` module wiring and standardize inter-module interfaces

---

### v1.0 for App Developers

* greatly simplify module wiring so that `app.go` is much more concise
* inter-module communication based on generated `Client` interfaces should greatly simplify `app.go`

---

### Building a blockchain should be really simple


```go
app := NewApp([]Module{
  bank.Module{/* some settings */},
  gov.Module{/* some settings */},
  staking.Module{/* some settings */},
})
app.Start()
```

---


## Stargate + Protobuf: the path to stable client interfaces
* Stargate introduced protobuf which provides a paradigm for evolving schemas while maintaining backwards compatibility
* `buf check breaking` is run against every commit to master to prevent breaking changes to `v1` proto files
* ensures that _old_ clients can work with _new_ modules

* clients have a single interface for interfacing with Cosmos SDK chains that has great cross-language support
* _and_ protobuf allows us to evolve schemas without breaking client code
* `buf check breaking` is run against every commit to master to prevent breaking changes to `v1` proto schemas


---

## Post-Stargate Client Priorities (Protobuf)
* stabilizing proto files as `v1` (they're currently mostly `v1beta1`)
* proto files for individual modules will likely reach `v1` before the SDK and when they do clients can expect stability
* protobuf code-generation improvements - better handling of `Any`s and custom types

---

## Post-Stargate Client Priorities (Tx)
* `Tx` API needs to be stabilized
  * need to design `SIGN_MODE_TEXTUAL` to fully replace Amino JSON
  * improve multisig signing UX
  * public key address format

---

## Stability for Module Developers
* simplify and stabilize core `types/` package
    * remove things that don't belong there
    * rethink `sdk.Dec`
* simplify and improve the `AppModule` interface and inter-module wiring

---

## Inter-module communication

In Stargate, queries and transaction messages are defined as protobuf `service`s:

```proto
package cosmos.bank;

service Msg {
  rpc Send(MsgSend) returns (MsgSendResponse);
}
```

---

## Inter-module communication

This defines a golang interface:

```go
type MsgClient interface {
  Send(context.Context, *MsgSend) (*MsgSendResponse, error)
}
```

Clients already use this interface to build transactions.
Modules will be able to use this same interface to talk to other modules in an authenticated way.

---

## Building a module should be really simple
1. write proto definitions for clients
2. generate code
3. implement generated interfaces
4. register services in single entry-point

---

## Stability for App Developers

* greatly simplifying `app.go`
* moving most of the inter-module wiring to modules
* app developers should mainly need to worry picking modules and setting the right config params

---

## Building a blockchain should be really simple

1. choose modules
2. set config params
3. start app

```go
app := NewApp([]Module{
  bank.Module{/* some settings */},
  gov.Module{/* some settings */},
  staking.Module{/* some settings */},
})
app.Start()
```

---

**We hope this roadmap brings us one step close to realizing the Cosmos vision**

**If you have comments, questions, etc. please post in https://github.com/cosmos/cosmos-sdk/issues/7421**

---

# Thank you!