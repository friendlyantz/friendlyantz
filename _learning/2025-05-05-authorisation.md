---
excerpt: authorisation concepts
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - learning
  - security
  - authorisation
---
# Models [^1]
- [ ] DAC - Discretionary 
	- [ ] every user has a permission to another resource - similar to linux file system
	- [ ] Problems: has to many permission/role objects, which can be mitigated by joining users into groups for example
- [ ] MAC - Mandatory
	- [ ] Further reading:
		- [ ] Bell & LaPadula (mid-level sec)
		- [ ] Chinese Wall
- [ ] RBAC - Role Based (i.e Kubernetes), Zero gem is clasic example in Ruby
	- [ ] still not flexible due to amount of roles
- [ ] ABAC - Attribute-based
	- [ ] adds context

just use PORO in Ruby [https://actionpolicy.evilmartians.io/](https://actionpolicy.evilmartians.io/)
# Resources
[^1]: [https://speakerdeck.com/palkan/rubyrussia-2019-welcome-or-access-denied](https://speakerdeck.com/palkan/rubyrussia-2019-welcome-or-access-denied)