# Qbix auth

An authentication protocol that's secure, private and compatible with nearly everything out there.

You can [read the overview](#overview) and [implement the spec](#specification).

# What does it do?

In a nutshell, it helps a user...
* authenticate with your site securely and easily without entering a password
* prove their ownership of an account on various sites
* securely manage their identity across many sites and devices
* share identities with friends in their private address books

Optional [social](https://github.com/Qbix/auth) extensions to this protocol improve:
* [ONBOARDING](#onboarding): get an instant personalized social experience on your site before they even authenticate
* [ENGAGEMENT](#engagement): discover all their friends already on your site, and the content they created
* [VIRALITY](#virality): invite friends to join their activity on your site in one click
* [RETENTION](#retention): receive notifications from your site on a device of their choice
* [SOCIAL](#social): get notified when people in their private address book join

# Why is it needed?

Throughout the world, people rely on using online services more than ever before, but the auth situation is one big mess, in several areas.

## Convenience

People today maintain user accounts on tons of different services, each with their own authentication mechanism. 

Some use [passwords](https://arstechnica.com/business/2012/03/passphrases-only-marginally-more-secure-than-passwords-because-of-poor-choices/), Passwords are annoying to remember as the number of sites grows, and each site has slightly different rules (capital letters, special characters, restrictions). To increase convenience, people [re-use passwords](https://xkcd.com/792/) and choose really [easy-to-guess](http://www.dailymail.co.uk/sciencetech/article-4125128/The-common-passwords-used-2016.html) ones, leading to a breakdown in security.

Some sites use [magic links](https://www.sitepoint.com/lets-kill-the-password-magic-login-links-to-the-rescue/) and use email and sms to recover passwords. A typical sign-up process involves providing an email address, checking for a welcome email, and clicking the verification link. Magic links require this same process to be repeated for every login. To add to this inconvenience, if the user's email ever changes, they have to update all their accounts.

Many apps upload their users' entire address book to the server, to find out if any of their contacts already use the service. While this is convenient, it runs many risks of compromising the privacy not just of users but of their contacts. Even large, well-known apps have been known to [send this information unencrypted](https://www.theverge.com/2012/2/8/2785217/path-ios-address-book-upload-ceo-apology), and in any case, the end result involves many companies storing [personally identifiable information](https://en.wikipedia.org/wiki/Personally_identifiable_information), which can be [breached](https://www.nytimes.com/2016/12/14/technology/yahoo-hack.html?_r=0) and [leaked](https://www.wired.com/2015/08/happened-hackers-posted-stolen-ashley-madison-data/) online for all to see.

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

# Overview

This section is non-normative. It's a high-level description of the various parts of the protocol, and the reasons for them.

At its core, a person posts identity claims on various accounts signed with their public keys. They then store the corresponding private keys using apps on their personal devices, inside the [secure](https://www.apple.com/business/docs/iOS_Security_Guide.pdf) [zone](https://en.wikipedia.org/wiki/ARM_architecture#TrustZone) of their trusted devices. Sites initiate UX flows using these apps to authenticate using [challenge-response](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication). Optional extensions allow the user to authenticate in other ways, issue identity and role certificates to contacts, etc.

## Identity

An identity claim can be hosted by any site at a url. It consists of text which contains
* the public key of a [private-public key pair](https://en.wikipedia.org/wiki/Public-key_cryptography)
* a timestamp
* which apps to use for authentication (optional)
* additional public keys for [identity conflict resolution](#compromise) (optional)
* any extra information (optional)
* and a digital signature generated with the corresponding private keys

Identity claims, if they are modified, can only append new information. Modifications might include adding new private keys or repudiating old keys, similar to the [keybase model](https://keybase.io/blog/keybase-new-key-model). Each modification must be signed with a majority of the previously listed public keys, to deal with [compromised identities](#compromise).

By hosting the identity claim at a given url, the site is understood to "endorse" the claim. *Even sites that do not support this protocol* can be used in this manner. Many well-known social networking sites, such as `facebook.com` or `plus.google.com`, have certain URLs where arbitrary content can only be submitted by the authorized user of the site. An example is the user's "about" page, but not their "timeline" page, since comments on posts can be a way for other users to contribute arbitrary text to the page. An identity claim appearing on one of those URLs constitutes a reasonable assumption that the owner of the account has posted it, although this assumption is specific to the specific site and its current policies.

Identity claims alone can already be used to show that the same entity who controls account X1 at site Y1 also controls account X2 at site Y2, because they include the same public keys (and have been signed with the same corresponding private keys). When such an entity [authenticates](#authentication) with your site, they can optionally reference URLs of various identity claims signed with those keys.

## Discovery

A particular person (or other entity) can post identity claims on various accounts at various sites. Each such claim may use the same public-private key pair, or a different one (see [authentication](#authentication)).

Identities may be public or private. If an identity claim is posted at a publicly accessible URL, it can be linked to from many places (such as articles the user writes, their public profile, or the user's [WebID url](https://github.com/solid/solid-spec#identity)). Such an identity claim can in principle be discovered and verified by anyone.

A certain level of privacy can be obtained if the identity claim is hosted at one or more obscure URLs, derived from a secret key [previously agreed upon](https://en.wikipedia.org/wiki/Out-of-band_agreement) by various participants. These URLs may also be used for [publishing and subscribing](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern) to messages, including identity claims, between various participants. The messages can be encrypted for their intended recipients. They can be hosted by [hubs](https://en.wikipedia.org/wiki/Hub_(network_science_concept)) or embedded in [blockchains](https://en.wikipedia.org/wiki/Blockchain) .

Discovery of private identities must be bootstrapped from address books, otherwise it's [turtles all the way down](https://en.wikipedia.org/wiki/Turtles_all_the_way_down). People maintain address books on their devices, with varying degrees of privacy depending on how this information is stored and synced. They can use these to connect with their personal contacts across different sites, in a *private and secure* manner. Here is how it works:

If person A has person B's email address, phone number, or other identifier in their address book, and vice versa, then they can derive the same secret key and discover each other on a particular site in this way. Since a person knows their own phone number, they can then compute, for example, `sha256(siteUrl + number1 + number2)` where `(number1, number2) = sort(numberA, numberB)`.

Normally, phone numbers have 15 digits or less, with US phone numbers typically being around 10 digits – a maximum of 10 billion possibilities. Hashing them on their own is useless, since the hashes can easily be reversed by the site on which they are stored. Using this scheme, however, A and B can find each other on a site, without revealing their phone numbers or other identifiers to the site.

## Authentication

Users store private keys in apps running on their private devices. The signed [identity claim](#identity) they post on a particular website (whether it is aware of this protocol or not) contains a list of apps (on various platforms) that this user has installed which can be used to verify their identity. These apps are identified by URLs which can resolve either to a server (such as `https://groups.org`) or an app running on a device (such as `groups://`). Your website can then redirect to this app with a challenge-response to authenticate the user with either the [secured oAuth 2 protocol](https://sakurity.com/oauth) or [the securelogin protocol](https://github.com/sakurity/securelogin).

Apps that store private keys and handle the challenge-response should have a way to be "locked", and require a passcode or biometric id to be "unlocked". The apps only handle challenge-response when "unlocked". They may be "locked" automatically when the mobile phone or tablet is locked, for example.

Private keys are stored in the [secure](https://www.apple.com/business/docs/iOS_Security_Guide.pdf) [zone](https://en.wikipedia.org/wiki/ARM_architecture#TrustZone) of the user's device, using operating system APIs such as the [MacOS Keychain](https://developer.apple.com/library/content/documentation/Security/Conceptual/keychainServConcepts/01introduction/introduction.html. If a computer supports multiple user accounts, the operating system would manage access to the private keys of the currently logged-in user.

Authentication of a session should be done only in the context of end-to-end encryption with a [key exchange algorithm](https://en.wikipedia.org/wiki/Key_exchange). Then, the session id cookie becomes a bearer token between the user agent and the web server, which is sent with every request.

During authentication or afterwards, the user may reference certain URLs of identity claims, to prove their control of certain accounts on various sites. The relying site may verify these claims server-to-server for public identities, or using [postMessage](#authentication-with-postmessage) for private identities.

## Authentication across apps

Sites which implement this extension to the Qbix auth protocol allow completely private identity claims. When using a standards-compliant user-agent such as a browser or a native mobile app, a consumer website ([relying party](https://en.wikipedia.org/wiki/Relying_party)) loads an iframe from a certain social networking site (identity provider) and requests information. The identity provider loads a document in the iframe, which is able to communicate with the relying party via [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) . The Javascript in this document is able to verify that the current user is already logged in (authenticated) with the identity provider, and also verify the domain of the relying party. It can then provide information to the relying party, if the current user has authorized it.

In fact, the identity provider *does not need to be a server*. A native app using WebViews, or even a [web app using Service Workers](https://www.youtube.com/watch?v=4uQMl7mFB6g), can [intercept HTTP requests](http://robnapier.net/offline-uiwebview-nsurlprotocol) and load their own HTML and Javascript for urls that begin with a certain prefix such as `https://groups.intercept/` . This Javascript can communicate with the native app (e.g. via a grandfather iframe) to fetch verify the logged-in user's information, and the sites they authorized to receive this information. They can then use postMessage the same as above, all the while keeping the identity provider completely client-based.

A relying party can authenticate a user using postMessage when it's available. Sometimes, for extra security, it may require using one of the device-based apps listed in the user's identity claim. The latter may be more desirable for banks since it guards against e.g. someone else leveraging an authenticated session in an identity provider that a user left unattended on a public computer.

## Authentication across computers

A user may securely authenticate sessions on other computers by any method which does not involve passwords. For example, a public computer may display a QR code which the user can scan with one of the authentication apps on their device. The code contains a challenge as well as the URL to send the response to. The authentication app then sends a request to the URL specified in the QR code over the internet, or if there is no wireless internet connection, it may display a code to the user to type, or communicate with the other device via bluetooth. Other approaches involve NFC, and so on.

This can be combined with the [Authenication across apps](#authentication-across-apps) extension to have the user sign into their account on `groups.org` (e.g. on a public computer) and then use that authenticated session to authenticate any relying party websites.

## Provisioning

*Private keys should never leave any device*. If the user wishes to use their identity on one device to bootstrap their identity on another device, entirely new private keys must be generated and these should be added to all the identity claims on all the relevant accounts of the user alongside the other keys.

Each device should store a list of accounts where the user has published identity claims, and this list should be updated via pub-sub as in the [discovery section](#discovery). This way, any device can be used to provision any new device. You can also provision other types of computers. It's recommended for users to maintain keys on more than one device, to deal with lost or stolen devices.

## Compromise

If one device is compromised (e.g. stolen), the others can be used by the user to log into their accounts and repudiate the keys stored on the compromised device. Along the lines of [NIST recommendations](https://www.nist.gov/itl/tig/back-basics-multi-factor-authentication), it is strongly recommended that the user maintain a way to log into the sites which is not dependent on having the device. This can be two-factor authentication using a password-protected authenticator app, or using a different device. For sites that do not support this protocol, it is usually a password or magic link sent to an email. However, these may be compromised if the device is unlocked and already has access to email and autofills passwords. In any case, the user simply has to be able to still log into their accounts after a device is compromised.

In each identity claim, the user may list one or more additional public keys for identity conflict resolution. If listed, the user must sign the identity claim with the majority of those public keys as well. This way, if one device is stolen, they can update their identity claims on all their other accounts by signing those requests with a majority of other devices and computers (servers, etc.) used for signing the identity claims.

If the user's list of identity claims is up to date on each device, this can be done automatically once the user authenticates with each computer. It's recommended that the user keep some of these computers on separate networks (e.g. servers, devices, etc.) or offline.

## Private Data

This extension allows the app to send various user data to the relying party, using a symmetric key to either encrypt it or sign it.

An authentication app may be running on a user's device or on a server. A [hybrid cryptosystem](https://en.wikipedia.org/wiki/Hybrid_cryptosystem) is used whereby the relying party generates symmetric keys, encrypts them with the user's public keys, and sends them to the app. The app stores these keys for each relying party, and uses them to sign or encrypt any data sent to the relying party via postMessage or oAuth. This allows the relying party's server, or anyone with the symmetric key, to verify the integrity of the data.

The hybrid cryptosystem allows much faster encryption (symmetric key encryption) to take place, which may make a big difference for authentication apps which run on servers and send out data for millions of users.

## Onboarding

Sites which implement the optional [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) extension to this protocol allow the display of personalized information to a logged-in user, without them even needing to authenticate with the site.

## Engagement

## Virality

## Retention

## Social

Users can get notifications delivered to their devices when a contact joins a new site, and publishes their identity claim on it. This identity claim may be encrypted for only certain other users to receive it.

A user's authentication app automatically fetches the identity claims and adds them to the user's address book, thereby creating a social network experience for the user. The user discover new sites that their contacts joined, which may represent various interests that their contacts have, places they visited, and so on.

They can assign various labels to their contacts, and share various content with them on different sites. The labels can act as roles for permissions, and they can send certificates to their contacts (via encrypted pubsub and notifications) allowing them to access content they post on various sites. Or, they may manage roles and permissions on the sites themselves (where the relations are more site-specific).

# Specification

## Definitions
