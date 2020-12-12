---
slideOptions:
  theme: solarized
  transition: 'fade'
  # parallaxBackgroundImage: 'https://s3.amazonaws.com/hakim-static/reveal-js/reveal-parallax-1.jpg'
---

## The Road to Cosmos SDK v1.0
### Aaron Craelius
### CTO, Regen Network 
<img src="https://cosmos.network/images/logos/cosmos-logo-black.svg" style="border:0; box-shadow:none" height="80"/><img src="https://i.imgur.com/XnmHAZ4.png"  style="border:0; box-shadow:none" height="160"/>

---

## <img src="https://i.imgur.com/XnmHAZ4.png"  style="border:0; box-shadow:none" height="250"/>
- Regen Network is launching a Cosmos Zone dedicated to ecological applications (Carbon Credits and beyond)<!-- .element: class="fragment" -->
- Since April 2020, Regen has been acting as the lead maintainer of the Cosmos SDK<!-- .element: class="fragment" -->

---


# What does 1.0 mean?

---

## 1.0 is a commitment to stability

---

### Maybe it's obvious, but why is stability important?
We can't build durable, long-lived systems without stability. We need a solid foundation to start building other layers.<!-- .element: class="fragment" -->

---

## 1.0 is a commitment to a great developer experience (DevX)

---

### Why is DevX important?
Blockchain is hard! A successful blockchain development platform __must__ have a strong focus on DevX to balance against the inherent complexity of developing distributed ledger technologies.<!-- .element: class="fragment" -->

---

## Stability & DevX for whom?


* Client Developers<!-- .element: class="fragment" -->
  * developers building user-facing apps that integrate with Cosmos SDK blockchains
* Module Developers<!-- .element: class="fragment" -->
  * developers building custom modules for Cosmos SDK blockchains
* App Developers<!-- .element: class="fragment" -->
  * developers building Cosmos SDK blockchains

---


## How Stargate moved us closer to 1.0:
* **Protobuf**: a standard multi-platform protocol for clients<!-- .element: class="fragment" -->
* **Live upgrades**: blockchains can gracefully upgrade without needing to start over from genesis<!-- .element: class="fragment" -->

---

## Stargate for Module Developers
**Simplified workflow for implementing a module:**
1. Write proto definitions<!-- .element: class="fragment" -->
2. Generate code<!-- .element: class="fragment" -->
3. Implement interfaces in generated code<!-- .element: class="fragment" -->
4. Wire up services in single `RegisterServices` entry point<!-- .element: class="fragment" -->

---

### 1. Write proto definitions

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

---

### 2. Generate code

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

---

### 3. Implement interfaces in generated code

```go=
func (k msgServer) Send(ctx context.Context, msg *types.MsgSend) (*types.MsgSendResponse, error) {
	...
}

func (k msgServer) MultiSend(goCtx context.Context, msg *types.MsgMultiSend) (*types.MsgMultiSendResponse, error) {
	...
}

```

---

### 4. Wire up services in single `RegisterServices` entry point

```go=
func (am AppModule) RegisterServices(cfg module.Configurator) {
	types.RegisterMsgServer(cfg.MsgServer(), keeper.NewMsgServerImpl(am.keeper))
	types.RegisterQueryServer(cfg.QueryServer(), am.keeper)
}
```

---

## Stargate for Client Developers

* client libraries in all languages<!-- .element: class="fragment" -->
* backwards compatible APIs: breaking change detection is run against every commit to prevent breaking changes to `v1` proto files<!-- .element: class="fragment" -->
* gRPC + auto-generated REST/OpenAPI support<!-- .element: class="fragment" -->
* easy to implement transaction signing given any off-the-shelf protobuf library<!-- .element: class="fragment" -->

---

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

---

## Automatic REST/OpenAPI Endpoints

![](https://i.imgur.com/uPCvGrY.png)

---

### Stargate for App Developers

**`app.go` is pretty massive 🤔**

![](https://i.imgur.com/2p1QHT3.png)

---

## Stargate was about setting a foundation
## But there's a lot work to do...<!-- .element: class="fragment" -->

---

### 1.0 Goals for Client Devs: Protobuf
* stabilizing proto files as `v1` (they're currently mostly `v1beta1`)<!-- .element: class="fragment" -->
* proto files for individual modules will likely reach `v1` before the SDK reaches `v1` and when they do clients can expect stability<!-- .element: class="fragment" -->

---

### 1.0 Goals for Client Devs: Transactions
* need to design `SIGN_MODE_TEXTUAL` to fully replace Amino JSON<!-- .element: class="fragment" -->
* improve multisig signing UX<!-- .element: class="fragment" -->
* standardized public key address format for new signature algorithms<!-- .element: class="fragment" -->

---

### v1.0 Goals for Module Developers

* protobuf code-generation improvements - better handling of `Any`s and custom types<!-- .element: class="fragment" -->
* simplify and stabilize core `types/` package<!-- .element: class="fragment" -->
    * remove things that don't belong there
    * better decimal support
* simplify and improve the `AppModule` interface and inter-module wiring<!-- .element: class="fragment" -->
* remove legacy code<!-- .element: class="fragment" -->

---

### Inter-module communication
Protobuf generates client golang interfaces:

```go=
type MsgClient interface {
	Send(ctx context.Context, in *MsgSend, opts ...grpc.CallOption) (*MsgSendResponse, error)
	MultiSend(ctx context.Context, in *MsgMultiSend, opts ...grpc.CallOption) (*MsgMultiSendResponse, error)
}
```

* Modules could talk to other modules using these interfaces<!-- .element: class="fragment" -->
* This could greatly simplify `app.go` module wiring and standardize inter-module interfaces<!-- .element: class="fragment" -->
* Authorization is handled automatically with this model<!-- .element: class="fragment" -->

---

### v1.0 Goals for App Developers

* greatly simplify module wiring so that `app.go` is much more concise<!-- .element: class="fragment" -->
* inter-module communication as described in the previous slide could be key to that<!-- .element: class="fragment" -->

---

**Building a blockchain from existing modules should be really simple**

<div class="fragment">
<strong>Maybe something like this?</strong>

```go
app := NewApp([]Module{
  bank.Module{/* some settings */},
  gov.Module{/* some settings */},
  staking.Module{/* some settings */},
  ...
})
app.Start()
```
</div>

---

### Independently versioned modules
* Maybe we can split base/core packages (like `types/` and `baseapp/`) and modules (`x/bank`, etc.) into separate go modules?<!-- .element: class="fragment" -->
* Then each module could have its own version and release cycles<!-- .element: class="fragment" -->

---

**If you have comments, questions, etc. please post in https://github.com/cosmos/cosmos-sdk/issues/7421**

---

**We hope this roadmap brings us one step close to realizing the Cosmos vision**


---

# Thank you!