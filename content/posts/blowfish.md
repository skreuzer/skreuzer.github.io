---
title: Using blowfish for password hashes in FreeBSD
date: 2011-12-24
tags: [freebsd]
---
FreeBSD uses sha512 to encrypted passwords for user accounts.
However, blowfish is available in all recent versions of FreeBSD
and its easy to change the default crypt method to use blowfish instead

In /etc/login.conf change

	:passwd_format=sha512:

to

	:passwd_format=blf:

and recreate the login capability database with the command `cap_mkdb /etc/login.conf`

Now have each user change their password. Start with your current login.

	$ passwd
	Changing local password for {current user}.
	new password:
	retype new password:
	passwd: updating the database. . .
	passwd: done

The second field in your password file, which is the cipher of the
passwords, should begin with $2 now which indicates the use of
blowfish. The command `grep ${USER} /etc/master.passwd | cut -d: -f2`
can be used to verify the change.

Then in /etc/auth.conf change

	#crypt_default = md5 des

to

	crypt_default = blf md5 des

All new users you now create with adduser will now have their password
encrypted in Blowfish.
