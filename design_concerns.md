# Design Considerations for Smart Containers.

SC should have at least these functional aspects:

1. Help user to navigate information capsuled or connected in smart container.
    * Detect author information (i.e. from ORCID).
    * Detect user information (i.e. from ORCID).
    * Detect related publication information.
    * Detect host machine information.
2. Instruct user to preserve a computational experiment.
3. At some point of a computational experiment add steps and preserve as a new experiment.



Read provenance of the computational experiment (normally read-only, at some special case, may need to modify forcefully) read provenance of underlying workflow specs(ORE, research objects).

Read computational environment (requirements of provisioning, both software/hardware)
detect host machine software/hardware specs offer affordance/ select affordance when some requirements cannot fully be fulfilled detect output file.

Checksum completeness of the reproduced experiment.



For 2. I suggest that we make some rules of write dockerfile. Besides normal dockerfile instructions, there should be a part with provisioning requirements like what vagrant/puppet used to provision environment.
Author should separate computational activities from provisioning activities.

