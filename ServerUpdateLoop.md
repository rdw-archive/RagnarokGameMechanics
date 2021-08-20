# World Updates in Ragnarok Online

According to unverified sources, the server software is running a much faster update cycle than the popular emulators:

> [It uses] a different server refresh speed (aegis is faster)

> [Official servers] are not really coded in a very efficient way, you need a really really strong server just to get it to run at all. [They] run through everything in a 20ms interval ... on *Athena emulators we changed this to a 100ms interval (and often even slower for certain things), so it needs 5+ times less CPU power. Unfortunately that also makes implementation of certain things like firewall behavior impossible to replicate to 100%.

While these claims seem believable, I haven't yet verified their veracity. What I have been able to confirm (so far):

* In Hercules, AI updates take at least 100ms (see ``MIN_MOBTHINKTIME``)
* Similarly, elemental follower updates take at least 100ms (see ``MIN_ELETHINKTIME``)

Considering that AI updates will be a significant chunk of the processing done serverside, many differences in behaviour could be explained by that alone.
