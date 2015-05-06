I starting thinking about the Go package management earlier this year in Jan. (realizing I was 6 months to a year behind...), however in the past few months a lot of attention has been directed around the *Go Package Manager story*. That said, now seems like the time to add my thoughts to the discussion.

## GoPkgr: Steps Towards A Decentralized Package Manager ##

Outlined below are steps towards the Decentralized Package Manager in somewhat phases:

**I. General Management**

  * Interface Versioning (InfVer 1.0.0)
  * Centralized Dependency Trees
  * Comment Tags

**II. Decentralized Management (Block Chain Based)**

  * Decentralized Dependency Trees
  * Proof of Test
  * Inferred Versioning (InfVer 2.0.0)

**III. Private Management**

  * Private Dependency Trees

### gopkgr: The application ###

An application will be the main command line interface with the following concepts. It is a wrapper around `go get` at times. However it can also do things like interact with block chains or other things outside the scope of the built-in `go` tools.

### Interface Versioning (InfVer 1.0.0) ###

The idea is to use the public interface of a project as the version number of the project. When used with a Distributed Version Control System (DVCS) every commit will in fact be a different version. This can allow a 1:1 mapping of versions to commits, which can be easier to reason about when trying to create dependency trees.

**For example:**

      ...
      91.  commit: a3f563a3 == v1.2.3
      92.  commit: fc565c12 == v1.2.4
      ...
      110. commit: 56daa7bb == v1.2.12
      ...
      151. commit: 7ced800b == v1.3.0
      ...
      431. commit: 0abc6d45 == v2.0.0
      ...

With Interface Versioning the following can be determined at any point during the commit history:

   1. Commit 92 was a bugfix to commit 91
   2. Commit 110 is one of several bug fixes later
   3. Commit 151 is an additional public API interfaces added after commit 92 
    
    *It's ideal to always add new public API interfaces and not break existing ones, then later clean up the new interfaces with a new public API version.*
    
   4. Commit 431 broke backwards compatibility with an existing public API interface

This is very basic versioning and doesn't protect against a bug fix breaking compatibility and being caught automatically, that's the next logical step with InfVer 2.0.0.

### Centralized Dependency Trees ###

The idea is send a single request for multiple packages, that then can have a dependency tree established. They will be proxy and vendor imported packages much like `gopkg.in`. 

     import "gopkgr.example.com/github.com/artktec/go-lib1"
     import "gopkgr.example.com/github.com/example/go-lib2"
     import "gopkgr.example.com/bitbucket.org/example/go-lib3"
     
Can have the dependency tree determined by being presented to gopkgr.example.com as a group:

     go get "gopkgr.example.com/:github.com/artktec/go-lib1:github.com/example/go-lib2:bitbucket.org/example/go-lib3"
     

Use the `gopkgr` tool to present them as a group to https://gopkgr.example.com for which dependencies will be resolved and each package will downloaded properly vendored within the `gopkgr.example.com` domain.

### Comment Tags ###

The idea is to mimic [StructTags](http://golang.org/pkg/reflect/#StructTag) which are used to augment structs; but these instead would hide behind a comment to augment the import statement. This should be inline with the same concept that is being used to define conical import paths starting in Go 1.4. By using a comment will prevent any backwards incompatible changes and keep the Go 1 compatibility contract.

**For example:**
   
    import "gopkgr.example.com/github.com/artktec/go-lib1" //`ver:"1.0.*"`
    
The Comment Tag (``//`<tag>` ``) immediately follows the import statement it is attached to. A Comment Tag can be parsed using the same parsing logic as Tags on structs.

With Comment Tags there is no need for a seperate file to determine dependencies.

### Decentralized Dependency Trees ###

The idea is to store the dependency trees (which is essentially can be thought of as the .lock file) of each library on a block chain. This will give the benefits of allowing **Proof of Tests** to determine the best dependencies well before they are needed.

The block chain can be distributed and give a single conical view of a library and it's dependencies at any point in time.

### Proof of Tests ###

The details are still being worked out on this. The idea is that there will be testers along with miners that protect the block chain. Testers can be thought of as specialized miners that have the necessary protections in place to run arbitrary code (as tests) but also get a higher reward.

### Inferred Versioning (InfVer 2.0.0) ###

The idea is to use the results of tests (Proof of Tests) of public interfaces to determine the version number. This is the next step after plain Interface Versioning. When used with a DCVS, each commit will be mapped to a version number as well. The number is not strictly based on the public interfaces, but rather the inferred interoperability of the versions.

This can give some protection against backwards compatibility issues when determining dependency trees. It can provide a high level of confidence of moving libraries to different versions will work based on verified public tests. Especially if more than one of the same library exists in a full project, this can show the best upgrade path.

### Private Dependency Trees ###

The idea is to create a fork in the block chain for which you control. This means that only you can use a private repo for all of the benefits of the public decentralized repository, while still getting updates from other external libraries. This can be used for private repos.


## Working code ##

Some of the ideas presented above have not been fully fleshed out. But I don't believe they are all completely pie-in-the-sky beyond reality ideas.

I have written some small code that can PoC some ideas:

**Comment Tags:** I have a simple parser based off of the go/ast and related packages.

**Centralized dependency trees:** I have a simple server running code that can take the specialized `go get` request and return meta tags that point to the proper place.

**InfVer 1.0.0:** I have code that can Hash public interfaces and determine a version based on the hash comparisons

**The Block Chain:** I have a simple block chain implementation (no networking or queuing code) that can create and mine (slowly) block chains. The next step is saving the specialized transactions to the block chain.

## Summary ##

None of the code above really works together. There will need to be some effort to do this. However before I go too much further down the path of making the working demo. I wanted to solicit feedback, to see what others thought.
