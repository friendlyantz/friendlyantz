---
excerpt: 
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - learning
  - authorisation
  - webauthn
  - authentication
---
# Authentication
## WebAuthn

[my app using it https://strivo.me/registration/new](https://strivo.me/registration/new)

or

[official demo](https://webauthn.cedarcode.com)

 [rails 8 demo app repo](https://github.com/cedarcode/webauthn-rails-demo-app)

[commit how i implemented(further cleanup /daisy ui conversion was later added)](https://github.com/friendlyantz/take-on-me/commit/ba68ad632bac7117dcdfb87f1eb68b5251f4d435 )

https://github.com/cedarcode/webauthn-ruby
# Authorization
## Models [^1]
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