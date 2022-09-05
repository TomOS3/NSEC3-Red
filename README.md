# NSEC3-Red

## Abstract
In this experiment some new subdomains have been identified by collecting and cracking NSEC3 hashes that were not yet identified by OWASP Amass.

## Introduction
Multiple tools and sources are available for reconnaissance by subdomain enumeration. Here you can find a nice overview:
https://pentester.land/cheatsheets/2018/11/14/subdomains-enumeration-cheatsheet.html
One of the tools, Amass by OWASP (https://github.com/OWASP/Amass), combines several online sources.

## Background
NSEC3 is used for prove of non-existence. A hash is created for every subdomain and NSEC3 records (1, 2 or 3, read this if you want to know why: https://www.sidnlabs.nl/downloads/Z0FQoNNjTRSwc1LTMrF7gQ/c2f8431b9b995c5671347d6a239a6b9b/wp-2011-0x01-v2.pdf) are returned for DNS requests for non existing domain names proving that there are no hashes between hash "x" and hash "y". If we can collect enough NSEC3 records for the zone, we can reconstruct a closed chain (cycle) to check if we have found all hashes for the domainnames in the zone.

## Research Question
Can we find subdomains by manually cracking NSEC3 records that are not known to Amass 3.19.2?
Can we do this in a way that it is not (too) obvious that we are collecting NSEC3 hashes?

## Related work
There are tools out there that:
- Collect NSEC3 hashes in an efficient way (not more DNS requests than necessary)
- Output hashes in a format that can be attacked using GPU power (e.g. Hashcat) 

## Methodology
Using a Python script, the hashes for a single domain are collected. We do this by taking an online available list (https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-110000.txt) of common subdomains as input for generating slightly different versions of these subdomains by mimicing typos. We store these hashes and generated subdomains in memory so that we can use these to select the next DNS request depending on the NSEC3 rows we collect along the way. This enables us to reconstruct the cycle by performing around 1 request per hash in practice.
The hashes are exported to Hashcat format. This enabled us to use GPU power to attack the hashes. In this experiment a combination of brute forcing and dictionary attacks (common subdomains list mentioned earlier, dictionary weakpass3) are used. Cracking took place in iterations, since a cracking action could unveil new subdomains.

## Results
All 600+ unique hashes were retrieved (the cycle was complete) 
~170 subdomains were only cracked using Amass urls.
~80 subdomains were cracked that were not known to Amass.

Major part of the cracked subdomains seem to come with technologies used, contain DNS wildcards, or tests urls. The latter might be particularly interesting.

The distance between hashes is smaller for domains that have a larger amount of subdomains. This means that additional hashes must be generated to have enough different hashes to be able to pick a hash in between every two of the hashes. During a quick test with 1500+ subdomains, a larger compilation of ~600000 subdomains was used and combined with different "typos" to generate even more variants. The smaller list mentioned before was not enough to find all hashes: only the last one could not be found :)

## Discussion
In theory, the value of the hashes of two domains can be very close to each other. In that case, it can be difficult to find the record that links the two hashes together (if there is any: if there is no hash in between there will be none). 
In that case, one can increase the amount of the precomputated hashes and rerun the program. If that does not help, one can try to find such a hash in a different way, or accept that the linking record is not there. You might miss other hashes in that case. 
Because the enormous amount of different subdomain guesses that has to be generated, it is not feasible to only generate subdomains that look like convincing typos. Using long delays between requests and using a large list of recursive resolvers might compensate for that to some extent. Of course, more convincing "typo" types could be introduced, like guesses containing words that are more related to the target or just based on known urls (e.g. based on Amass output).
The amount of retrieved subdomains seems, for the single domain, to be of little value to the attacker. One might doubt the added value for an attacker to perform the attack on NSEC3 if other attacks could lead to better results using less time and effort.

## Limitations
We mainly looked at just one domain. For other domains, we might find less or more subdomains using manual cracking in comparison to Amass.
There are some Amass sources that require (commercial) API keys. These features have not been used during this experiment, only a default Amass has been ran.

## Conclusion
Some new domain names have been identified by collecting and cracking NSEC3 hashes that were not identified by running a default OWASP Amass.
