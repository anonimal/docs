
Getting started in Katzenpost development
=========================================

This guide assumes you are familiar with golang,
git and some kind of Unix operating system environment.


Overview of our git repositories
--------------------------------

The following two repositories contain code that compiles to
executable programs:

* tools - Tools are programs that we use for testing and debugging.

* daemons - The Daemons repo is used to build the Katzenpost
  components such as mix client, mix server and PKI server. This
  repository has vendored dependencies, however for developement and
  testing we generally do not use vendored dependencies.

* mixnet_uprising - Repository for tracking open tasks for the
  Katzenpost mixnet framework.
  
* docs - All our documentation is here with the exception of our
  website. This includes our design specifications as well as this
  document you are currently reading.

* core - Core library

* server - Server library

* authority - Mix PKI library

* client - Experimental client library

* website - The Katzenpost website

* minclient - Minimal client library which is the basis for all
  other mix clients.

* mailproxy - High-level client library with optional reliability and
  optional SMTP and POP3 listeners.

* noise - The Katzenpost fork of flynn's golang Noise crypto library
  implementation which has the ablity to use the New Hope Simple
  post quantum hybrid key exchange for forward secrecy.

* nixops - NixOS automated configuration for Katzenpost server components.

* nixpkg - NixOS packages for Katzenpost components.

* bindings - Language bindings for Java and Python. STATUS: NOT finished.

* pykatzenpost - Python client library for Katzenpost.

* katzsim - Client simulator for load testing mix servers.

* katzenpost-android-mail - A Katzenpost client, based on K-9 Mail.

* pykatzen-auth - Optional Katzenpost server authentication module in python.


client and server internals
---------------------------

Katzenpost is NOT crash-only software. Everything has a proper
shutdown code path unlike many beginner golang examples on the
Internet. Struct types which act as worker goroutines MUST be a
composite struct type with the Worker type which is defined in our
"core" repository here:

* https://github.com/katzenpost/core/blob/master/worker/worker.go

Correctly implementing a composite Worker type means that your
code uses the Worker's Go method to spawn new goroutines and will
halt it's runtime loop upon receiving from the channel returned
by the Worker's HaltCh method. There are plenty of examples of this
in our code which you can learn from.


server internals
----------------

The Katzenpost server is essentially a software based router and as
such it utilizes three active queue management algorithms (AQM). These
queues are called the ingress queue, the mix strategy queue and the
egress queue. We utilize a computational model called SEDA or Staged
Even Driven Architecture where these three queues are pipelined together.

In the SEDA computational model there are several processing stages
which are pipelined. At each stage there is a thread pool of
workers which perform the computation for that stage. Between each of
these stages is an AQM which can drop work tasks and can have dynamic
load shedding properties so that performance degrades gracefully with
respect to increased work load.

If you'd like to learn more about the SEDA computation model we
recommend reading:

* "SEDA: An Architecture for Well-Conditioned, Scalable Internet Services",
  http://www.sosp.org/2001/papers/welsh.pdf

Currently Katzenpost only supports the Poisson mix strategy and
therefore the mix strategy AQM is implemented using a priority
queue. To learn more about the Poisson mix strategy you should read:

* "The Loopix Anonymity System",
  https://arxiv.org/pdf/1703.00536.pdf

* "Stop-and-Go-MIXes Providing Probabilistic Anonymity in an Open System",
  https://www.freehaven.net/anonbib/cache/stop-and-go.pdf

The Katzenpost server repository has several coding themes which you
should become familiar with before making a contribution. Besides the
composite Worker type mentioned above, it's important that the server
side not have any unbounded resource consumption such as spawning new
go routines.


Testing Katzenpost with Kimchi
------------------------------

Our tools repository contains a Kimchi our integration test tool
for Katzenpost mix clients, servers and PKI Directory Authority:

* https://github.com/katzenpost/tools/tree/master/kimchi

Currently Kimchi does not utilize a configuration file. You may need
to make minor code changes to Kimchi in order for it to test your new
code.  Kimchi does not test any code in the daemons repo. Instead it
provides an alternate main function which spawns many goroutines to
run each component of the Katzenpost system.


Making a code contribution
--------------------------

0. Meet the Katzenpost developers

   Chat with the Katzenpost developers on irc: #katzenpost on the OFTC
   network or reach out to us on our mailing list:
   https://lists.mixnetworks.org/listinfo/katzenpost

   It is a good idea to discuss your code change with us before
   investing your time in writing the code.

1. Write a specification document

   If your code change is complex or requires us to change any of our
   protocols you will need to first propose a draft specification
   document. You can do this by forking our docs repository, creating
   a new git branch with your specification document and then
   submitting a pull-request.

2. Document your feature addition

   Open a ticket to document your feature addition or code change using
   the repository's issue tracker.

3. Testing your code

   Your code should have unit tests. However you should also test your code
   using our kimchi tool.

4. Request code review

   Finally you can submit a pull-request for your code changes or
   additions. We will review your code. There may be several rounds
   of code reviews until the code is of sufficient quality to be
   merged.
