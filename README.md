# Qbix auth

An authentication protocol that's secure, private and compatible with nearly everything out there.

[Here is the spec](../blob/master/SPEC.md)

# What does it do?

In a nutshell, it helps a user...
* authenticate with your site securely and easily without entering a password
* prove their ownership of an account on various sites
* securely manage their identity across many sites and devices
* share identities with friends in their private address books

Optional [social](https://github.com/Qbix/auth) extensions to this protocol improve:
* ONBOARDING: get an instant personalized social experience on your site before they even authenticate
* ENGAGEMENT: discover all their friends already on your site, and the content they created
* VIRALITY: invite friends to join their activity on your site in one click
* RETENTION: receive notifications from your site on a device of their choice
* SOCIAL: get notified when people in their private address book join

# Why is it needed?

Throughout the world, people rely on using online services more than ever before, but the auth situation is one big mess, in several areas.

## Convenience

People today maintain user accounts on tons of different services, each with their own authentication mechanism. 

Some use [passwords](https://arstechnica.com/business/2012/03/passphrases-only-marginally-more-secure-than-passwords-because-of-poor-choices/), Passwords are annoying to remember as the number of sites grows, and each site has slightly different rules (capital letters, special characters, restrictions). To increase convenience, people [re-use passwords](https://xkcd.com/792/) and choose really [easy-to-guess](http://www.dailymail.co.uk/sciencetech/article-4125128/The-common-passwords-used-2016.html) ones, leading to a breakdown in security.

Some sites use [magic links](https://www.sitepoint.com/lets-kill-the-password-magic-login-links-to-the-rescue/) and use email and sms to recover passwords. A typical sign-up process involves providing an email address, checking for a welcome email, and clicking the verification link. Magic links require this same process to be repeated for every login. To add to this inconvenience, if the user's email ever changes, they have to update all their accounts.

## Security

Another major problem is security. 

Magic links, including two-factor authentication links, sent to mobile phone numbers can be intercepted by social engineeering of [company sales reps](http://www.eweek.com/security/nist-says-sms-based-two-factor-authentication-isn-t-secure) or [the user themselves](http://www.firstpost.com/business/password-recovery-scam-hackers-stealing-gmail-yahoo-mail-accounts-2299854.html).

[Even large sites](https://www.wired.com/2012/08/apple-amazon-mat-honan-hacking/) can have policies that allow the "change password" feature to be abused. At the very least, if your provider supports +extensions, try to have a different hard-to-guess email+extension@provider.com for each service you log into.

Some sites store passwords in plain text. Sometimes you can tell when their "forgot password" link emails you your actual password. [This leads to epic breaches of millions of passwords](https://arstechnica.com/security/2016/09/plaintext-passwords-and-wealth-of-other-data-for-6-6-million-people-go-public/), and since passwords are re-used, you can bet they're tried on your other accounts.

Until not too long ago, sites used to [innocently asked for your password](https://blog.codinghorror.com/please-give-us-your-email-password/) to important accounts. Many people provided them, simply trusting that no one would ever store the password and log in as them. This is the nightmare scenario, since you've probably got all your bank accounts and other major accounts linked to your email, and a simple "change password" email will let the attacker into all of them. That attacker may just be the site which [innocently asked for your email password](https://blog.codinghorror.com/please-give-us-your-email-password/).

To mitigate this, [oAuth](https://oauth.net/2/) was invented, and now many sites have oAuth buttons to let users authenticate with large providers like Facebook and Google, and authorize access to some resources. This is better, and a database breach in one site doesn't lead to compromised passwords all over the place. But you are still trusting Facebook and Google with your identity. They can shut you out at any time, so you can't log into any of your other websites. And if someone logs into your Facebook (which uses a password), then they can log in as you on other sites as well.

Also, because of its loose spec, oAuth is vulnerable to cross-site request forgery, session fixation attacks and many more things, so at least use a [better oAuth spec](http://sakurity.com/oauth).

## Decentralized

The web was designed to be decentralized. Today people carry mobile phones in their pocket, with great security and privacy features built in. But they still rely on huge, centralized server farms to manage all their authentication. They trust their identity and their data in the hands of large corporations and remote services ["in the cloud"](http://ascii.textfiles.com/archives/1717). What happens to all of it if those services are discountinued tomorrow? What if your account is shut down tomorrow due to a misunderstanding? If your ability to use all your accounts relies on your relationship with one particular organization, they have a lot of power in that relationship. Why give your personal power away?

