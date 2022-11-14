# What is P4TC you ask? 

A **<u>Scriptable</u>** *Hardware offload of P4 MAT(Match Action Tables) and their Control* using the kernel TC infrastructure.

Both user space *Control* and Kernel *Datapath* are scripted (<u>not compiled</u>).

This means (once the P4TC iproute2 and kernel part are upstreamed) there is no kernel
or user space dependency for any P4 program that describes a new datapath.
i.e introducing a new datapath is a matter of creating bash scripts for the relevant
P4 program with no kernel or userspace code change. while these bash scripts can be
created by hand, it is simpler to use the *P4C* compiler to generate them (See Workflow
for details).

## The challenges with current Linux offloads

The Linux kernel already supports offloads today for several TC classifiers including but
not limited to *flower*[ref1] and *u32*[ref2].

Flower has become quiet popular and is supported by a wide variety of NICs and
switches[ref1, ref2, ...].

The diagram below illustrates a high level overview of TC MAT runtime offload for
the flower classifier (all the other offloadable classifiers have the same
semantics).
The example demonstrates a user adding a table entry on the ingress of device *eth0*
to match a packet with header fields *ethernet type ip, ip protocol SCTP and
destination port 80*. Upon a match the packet is *dropped*.
Observe that the match table abstraction exists both in hardware and in the kernel;
the user gets to select whether to install the entry in the kernel, hardware or both.
In the illustrated example below, the entry skips the software table (keyword *skip_sw*)
and is installed only in the hardware table. To install the entry only in the
kernel/software table the keyword *skip_hw* is used. When neither *skip_sw* or *skip_hw*
is specified the kernel attempts to install the entry both in s/w and h/w (it may fail
on the hardware side if there are no more resources available to accommodate the table entry).

![TC Flower Offload](./images/why-p4tc/tc-flower-offload.png)

Note: A wide variety of other kernel based offload approaches exist including
switchdev[refxx], etc. XXX: More text later.

### So what is wrong with current kernel offload approaches?

While the open source community development approach has helped to commoditize offloads,
and provide stability it is also a double edged sword; it comes at a cost of a slower
process and rigidity of features.

Often the hardware has datapath capabilities that are hard or impossible to expose
due to desire to conform to available kernel datapath abstractions.
Extending the kernel by adding new extensions takes an unreasonably long time to
upstream.  But even with already exposed datapath frameworks, such as the TC MAT
architectures, adding small extensions is non-trivial due to the upstream process.
As an example, when an enterprise consumer requires a simple offload match to extend
the tc *flower* classifier that is to eventually be supported by a distro vendor such
as RH, Ubuntu, SUSE etc it could take multiple years for such a feature to show up in
the enterprise consumer's network.
Adding a basic header field match offload (as trivial as that may sound) requires:  

1. patching the kernel flow dissector,

2. patching the tc flower user space code,

3. patching the tc flower kernel code

4. patching the driver code 

5. convincing the subsystem owners to agree on the changes. 

Adhering to such a workflow and eventually be able to deliver a required feature
calls out not only for perseverance but also above average social and technical skills
of the developers and associated management.

Simply put: The current kernel offload approach and process is not only costly
in time but also expensive in personnel requirements.

These kernel limitations have created a trend to bypass the kernel and move to user
space with solutions like DPDK. Unfortunately such moves are not good for the consumer
since they are encumbered with vendor-specific APIs and object abstractions.
OTOH, the kernel provides a _stable, well understood single API and abstraction_ for
offloaded objects regardless of the hardware vendor.

Due to supply chain challenges (which were exercabeted by the COVID pandemic)
the industry is trending to a model where consumers purchase NICs from multiple
vendors. Clearly a single abstraction here is a less costly approach and the kernel
approach stands out.

Network datapath deployments tend to be very specific to the applications they serve.
But even when serving similar applications, two different organizations may end up
deploying different datapaths due to organizational capabilities or culture. IOW, there
is no one-model-fits-all network datapath.
Typically this desire translates into request for just this "one feature" that perhaps
nobody else needs. These `one-offs` are abundant and the current upstream process does
not bode well for such requirements due to the process requirements.

### So why P4 and how does P4 help here?

P4 is the *only* widely deployed specification that is capable of defining datapaths.

It is quiet capable of providing a wide range of datapaths so much so that
large NIC consumers such as Microsoft (XXX: ref to DASH) and others have opted
to specifying their hardware datapath requirements with P4. The NIC vendors are
then tasked with delivering hardware that matches the provided P4 program.

From this perspective:
Think of a P4 program as an abstraction language for the datapath i.e
it describes the *datapath behavior*.

While P4 may have shortcomings it is the only game in town and we are compelled enough
to add support for it in the kernel.

### So where does P4TC play?

A major objective of P4TC is to address all the challenges mentioned earlier in the
current kernel offload approach.

For starters: P4 allows us to expose hardware features using a custom defined datapath
without dealing with the consequence of constrained kernel datapaths. This provides
us with an opportunity to describe only the features we need in the hardware and no
more. P4TC provides the datapath semantics that expose P4 abstraction.

From a P4TC perspective, the hardware implementation does not have to be P4 capable as
long as it abstracts out the behavior described in a specified P4 program. The user/control plane is interfaces with the (P4) abstraction of a pipeline and how the packet flows across MATs - and it is upto the vendor's driver to transform that abstraction and its associated objects to its hardware specifics.

To address the long process challenge mentioned earlier:
P4TC provides stability by outsourcing the datapath abstraction semantics to P4
while maintaing the well understood offload semantics provided by TC.
P4TC also introduces user and kernel independence for additional extensions to the
MAT architecture by using the concept of **scriptability**.

#### Sorry, what is *scriptability* again?

Ok, there is nothing magical about ability to script datapath computations in
the TC world. 

For the TC savy folks:

 - Think about the tc *u32* classifier which, as you know, can be taught how to parse
   and match on arbitrary packet headers with zero kernel or user space changes.

 - Think of the *pedit* tc action which, as you know, can be taught to edit or compute
   operations based on arbitrary packet headers with zero kernel or user space changes.

 - Think a programmable *skbedit* tc action where you can Read/Write from/to arbitrary packet
   metadata with zero kernel or user space changes.

Next:
…Think of all those designed from day 1 with intention to define arbitrary datapath
 and associated processing as defined by P4.
 ⇒ That is what P4TC is about….

For the non-TC savy but P4-savy folks: P4TC intends to solve the upstream process bottleneck
by allowing for creating shell scripts which contain TC commands that describe the P4
behavior with zero kernel or user space changes.

With that being said: The P4TC solution will go beyond traditional P4 in the focus for
network programmability - we hope that we can grow the network programmability paradigm
by marrying Linux and P4 and that some of these experiences can be brought into P4 proper
over time.

#### So how would a P4TC datapath look like at runtime?

XXX: Attach some diagrams here.

