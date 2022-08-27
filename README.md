# NSEC3-Red

## Abstract
In this experiment some new subdomains have been identified by collecting and cracking NSEC3 hashes that were not yet identified by OWASP Amass.

## Introduction
Multiple tools and sources are available for reconnaissance by subdomain enumeration. Here you can find a nice overview:
https://pentester.land/cheatsheets/2018/11/14/subdomains-enumeration-cheatsheet.html
One of the tools, Amass by OWASP (https://github.com/OWASP/Amass), combines different online sources.

## Background
NSEC3 is used for prove of non-existence. A hash is created for every subdomain and NSEC3 records are returned for DNS requests for non existing domains proving that there are no hashes between hash "x" and hash "y". If we can collect enough random NSEC3 records for the zone, we can reconstruct a closed chain (cycle) to check if we have found all hashes for the domainnames in the zone.

## Research Question
Can we find subdomains by manually cracking NSEC3 records that are not known to Amass 3.19.2?
Can we do this in a way that it is not obvious that we are collecting NSEC3 hashes?

## Related work
There are tools out there that:
- Collect NSEC3 hashes in an efficient way (not more DNS requests than necessary)
- Output hashes in a format that can be attacked using GPU power (e.g. Hashcat) 

TODO

## Methodology
Using a Python script, the hashes for a single domain are collected. We do this by using an online available list (TODO) of common subdomains as input for generating slightly different versions of these subdomains by mimicing typos. We store these hashes and generated subdomains in memory so that we can use these to select the next DNS request depending on the NSEC3 rows we collect along the way. This enables us to reconstruct the cycle by performing around 1 request per hash.
The hashes are exported too Hashcat format. This enabled us to use GPU power for attacking the hashes. In this experiment a combination of brute forcing and dictionary attacks (common subdomains list TODO, weakpass3) are used. Cracking took place in iterations, since a cracking action could unveil new subdomains.

## Results
TODO

## Discussion
In theory, the value of the hashes of two domains can be very close to each other. In that case, it can be difficult to find the record that links the two hashes together (if there is any: if there is no hash in between there will be none). 
In that case, one can increase the amount/length of the precomputated hashes and rerun the program. If that does not help, one can try to find such a hash in a different way, or accept that the linking record is not there. You might miss other hashes in that case (at least in theory). 

## Limitations
We looked at just one domain. For other domains, we might find less or more subdomains using manual cracking in comparison to Amass.
There are some Amass sources that require (commercial) API keys. These features have not been used during this experiment, only a default Amass has been ran.

## Conclusion
Some new domain names have been identified by collecting and cracking NSEC3 hashes that were not identified by running a default OWASP Amass.
