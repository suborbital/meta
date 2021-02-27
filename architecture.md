# Suborbital Architecture
This page is a general overview of the architecture of Atmo, and some notes about the architecture of the projects that comprise it.]

![Atmo Architecture](https://user-images.githubusercontent.com/5942370/109400927-8eb06400-7919-11eb-8ac7-45c11d83707d.png)

As seen in the diagram, Atmo is built by combining Vektor, Grav and Reactr.
- Vektor provides the HTTP server and all related functions
- Grav provides an in-process message bus as well as network meshing capabilities
- Reactr provides a "cluster" of compute comprised of workers executing Runnables (WebAssembly modules)

The Atmo Coordinator is responsible for taking incoming requests and processing them by executing the handlers described in the application's Directive. The Coordinator sends messages using the Grav bus, and the Reactr scheduler listens for those messages to execute Runnables.

When a Runnable Bundle is loaded, the Coordinator registers all of the WebAssembly modules with Reactr as Runnables, and configures the Grav and Reactr to route messages for each of those Runnables correctly. Reactr's integration with Grav allows it to send function results back to the coordinator as message replies.

The Coordinator is also responsible for generating a Vektor route group that encompasses all of the resources described in the Directive, and Atmo mounts that group to the Vektor instance on startup.

A 'coordinated request' object is created for each new HTTP request, which contains a network-serializable version of the request itself, along with the request state key/value map which stores the output from each function called during execution. The Coordinator is responsible for keeping request state up to date and satisfying the desired state described by the 'with' clause in the Directive.

Atmo is designed for Bundles to be hot-swappable, and as such the Coordinator has been built to eventually allow switching the "active bundle" on demand, but that has not yet been implemented.