---
title: Cryptography 1 - History and Basics
date: 2022-10-25
categories:
- cryptography
- mathematics
author:
- fnord
math: true
---
# Cryptography
## Part I: History and Basics
Hi everyone, Fnord here.
This post is the first in a small series on cryptography, written in collaboration with [CRTified](https://github.com/CRTified). For the first post, we'll mainly focus on the history of cryptography and set some basic ground rules. We'll also talk about what "crypto systems" are and take a look at a simple method of encrypting texts.

## Table of Contents
* History
* Notation/Ground rules
* What are crypto systems?
  * Symmetric
    * Caesar Cipher - a simple symmetric encryption
  * Asymmetric

### History
_Alt title: "what is cryptography anyways?"_
Don't worry, I'll spare you the `n + 1`[st|nd|th] detailed reiteration of things you can find on [Wikipedia](https://en.wikipedia.org/wiki/Cryptography), but let's at least recap a few major points:
* For as long as recorded history goes, people have felt the need to hide crucial or personal information from others. Earliest examples of what could be considered cryptography date back to Ancient Greece, India and Persia.
* Most classical crypto algorithms relied on either swapping the order of letters or the letters themselves, up until the 15th century when so-called polyalphabetic ciphers introduced more than one possible substitution per letter. We'll talk about those in another post.
* In the 19th century, specifically in 1883 with Auguste Kerkhoffs stating his famous principle "the enemy knows the system", the crypto world finally recognized that security _must_ come from the encryption key, not the encryption method.
* Computers kick-started modern cryptography, because suddenly it became very easy to perform complex operations very quickly. They also moved cryptography from being something that mostly military personnel and intelligence agencies were concerned with into the academic and public domain.
* The age of modern computing, with its ever-increasing amount of computational power (quantum computing, anyone?) poses new problems and threats to proven approaches.

"But where does he want to go with this?", you may want to ask. At this point, cryptography is something that concerns us all. It's in our phones, our banks, our computers and even in our refrigerators and ceiling lamps (hopefully). And I think it's important to understand what's going on. To a degree at least. And to understand where we stand and where we're going, we need to understand where we come from. If you take anything from this paragraph, let it be "the enemy knows the system" - never trust complexity as a security device.

And that's not even all there is to it! If we broaden the picture and take a look at the field of crypto**logy**, we'll find all kinds of useful applications. For example:
* Defending against attackers (cryptography, which this post mainly focuses on)
* Attacking secure systems (cryptanalysis, which aims to overcome cryptographic security measures by analyzing them and finding useful weaknesses)
* Security protocols, which provide a secure link between two interacting parties

### Notation/Ground rules
Before we begin, we need to agree on a few things. The first of which is "how do we actually write this down?". To keep things consistent across two authors and a number of posts, we'll define the most important details right at the start.

#### Names
Examples in the realm of cryptography always rely on names to illustrate who is interacting with whom. The most commonly used names are:

**Alice**
Alice is our "person A". She either pens the original message that starts a (hopefully encrypted) conversation or is the one that states our general problem.

**Bob**
Similarly, Bob is "person B" and usually the one to interact with Alice.

**Eve**
Eve is an **e**avesdropper that tries to listen in on whatever it is Alice and Bob are discussing.

**Mallory**
Mallory is an attacker that relies on **m**alleable algorithms. Don't worry, we'll discuss this when the time comes.

**Oscar**
Oscar is our generic **o**pponent and, similar to Eve, tries to interfere with the communication between Alice and Bob. He might occasionally show up for the sake of flavour.

We may at times need to introduce additional parties. For the sake of simplicity, all attackers will use names that start with the latter that suits their attack vector best (so a second eavesdropper might be called Eric and a second generic opponent might be called Olivia). All other actors will draw from the remainder of the alphabet.

#### Notation
When talking about cryptography, especially when doing to in theory, there is not really a way around using math one way or another. We will try to focus on "concept first". This means that we'll introduce math as necessary, explaining new symbols as they come into play instead of building a giant glossary in this post.

> Mathematical definitions will be put in quotes.

\[Mathematical \hspace{1mm} notation \hspace{1mm} will \hspace{1mm} look \hspace{1mm} like \hspace{1mm} this\]

Also, we'll sometimes use examples that provide a little more (mathematical) detail to illustrate our points. If we do, we'll follow these guidelines so you can follow along:
* $\textsf{Cryptographic Information (plain/cipher texts, keys, ...) is written using this special font}$. This applies to entire sentences as well as single letters and words.
* $\textsf{Algorithms will also be written in this style}$
* All examples will assume the standard latin alphabet of A to Z
* We'll use uppercase letters only

#### Glossary
There's a few words you'll regularly come across when talking about cryptography. The following list will also only provide a first set of basics and will most probably be expanded as we go along.
|word         |meaning                                                                                                          |
|-------------|-----------------------------------------------------------------------------------------------------------------|
|alphabet     |Set of characters (letters, numbers, symbols, ...) used to form texts. Not necessarily a conventional alphabet.  |
|plain text   |Original message, unencrypted and readable by anyone.                                                            |
|cipher (text)|Encrypted message, unintelligible and (hopefully) safe to pass on.                                               |
|key (word)   |Phrase or word used to start the encryption algorithm.                                                           |
|encrypting   |Using a cryptographic algorithm to convert plain text into a cipher.                                             |
|decrypting   |Using a cryptographic algorithm to recover the original plain text from a cipher.                                |
|attack       |Attempt to decrypt or modify a cipher text not intended for the attacker.                                        |
|breaking     |Finding a way of quickly solving a cryptographic problem or invalidating at least one of its goals by any means. |

Now, with the preamble done, let's get to the matter at hand!

### What are cryptosystems?
Simply speaking, a cryptosystem is a category of cryptographic methods that follow the same basic principle. Cryptosystems themselves usually belong to one of two categories: symmetric or asymmetric. Each cryptosystem has its own specific strengths and weaknesses and means of being attacked. Before we continue, we will formally define what a cryptosystem is in mathematical terms, because we'll refer to a lot as we go on. So buckle up, this is going to be a bit of a read.


#### Definition: cryptosystem
Mathematically speaking, a cryptosystem needs 6 components to be considered complete:

* A set of plain texts
* A set of cipher texts
* A set of cryptographic keys
* A key generation function
* A set of encryption rules using the above keys
* A set of decryption rules using the above keys

(Later on we'll also have to further detail some of them, but for now, they'll do.)

A "set" simply refers to any number of elements - imagine them like a magical bag that may potentially never be full.

Usually, the above criteria are represented by symbols grouped into something called a tuple, so we don't have to write the entire list each time. For these posts we'll use the following symbols:

> * $\mathcal{P}$ to represent the plain texts
> * $\mathcal{C}$ for the ciphers
> * $\mathcal{K}$ for the keys
> * $\mathsf{KGen}$ to sum up key generation
> * $\mathsf{Enc}$ to sum up the encryption rules
> * $\mathsf{Dec}$ for the decryption rules


And finally, to group them together we'll use brackets to represent them as a tuple:
> $(\mathcal{P, C, K}, \mathsf{KGen, E, D})$

By the way, "tuple" tells you that all elements within the brackets always have to be written in that very order (we call this an "ordered list").

To be a crypto system not just in name, there's one more rule we need to establish and that is how encryption and decryption work as mathematical functions. Using the above notation, the following must apply to any crypto system:
> * For every key $K \in \mathcal{K}$, there must be an encryption rule $e_k \in \mathcal{E}$ and a decryption rule $d_k \in \mathcal{D}$.
> * $e_k$ is defined as a function $e_k: \mathcal{P} \rightarrow \mathcal{C}$
> * $d_k$ is defined as a function $d_k: \mathcal{C} \rightarrow \mathcal{P}$
> * For all plaintexts $x \in \mathcal{P}$, $d_k(e_k(x)) = x$ must apply

Let's break this down. What it basically says (in math terms) is that each key in the system must be made in a way that allows you to encrypt and decrypt with it. "Encryption" is a function that creates a cipher text out of a plain text, "decryption" goes the other way around and creates _a_ plain text from a cipher text. And to be useful for its intended purpose, applying a decryption function on a text that has been encrypted using an encryption function from the same crypto system _must_ restore the original plain text (a property we call "correctness").
The first requirement also introduces an important notation: $K \in \mathcal{K}$ tells us that any key that is held within $\mathcal{K}$ can be represented by the letter $K$. Likewise, plain texts ($P \in \mathcal{P}$) and cipher texts ($C \in \mathcal{C}$) can be generally represented using letters.

The last requirement can be implemented in one of two ways and usually makes the difference between what's called a "symmetrical" or an "asymmetrical" crypto system. Let's look at the definitions for these two.

#### Symmetrical crypto systems
A symmetrical crypto system is defined as a system where the same key that is used to encrypt the message also decrypts the cipher. This makes them pretty easy to use and fast, at a price. For example, you can't (or at least shouldn't) share the same key with multiple parties, unless you want to share information with all of them. And you'll most likely have to come up with new keys on a regular basis in a sort of arms race between you and Oscar/Eve. Because they will try to get hold of your current key. And if they ever do, a large portion of your communications will be visible for them.

##### Caesar cipher
You most probably know this one, maybe by another name. The story goes that [Gaius Julius](https://en.wikipedia.org/wiki/Julius_Caesar) needed to come up with a means of securely getting military orders to his generals. And what he came up with is a solution as simple as smart, considering the resources he reasonably might've had at hand: he shifted the letters in the alphabet. The most commonly known representation of this is that of a rotating disk marked with two alphabets. This disk allows you to quickly shift the letters as needed.

//PLACEHOLDER: image of Caesar cipher disk

The key in this case is the amount of letters you shift. Assuming that $\textsf{A}$ can be represented by the number $\textsf{0}$, $\textsf{B}$ by the number $\textsf{1}$ and so on (you can see where this is going), you can for example use a key of $\textsf{5}$ to tell the recipient of your message that you've replaced all $\textsf{A}$s ($\textsf{0}$) with $\textsf{F}$s ($\textsf{5}$), all $\textsf{B}$s with $\textsf{G}$s, etc...
Side note: you'll see why we start counting at $\textsf{0}$ in the example below. It just makes life a little easier.

To decrypt the message, all your recipient has to do is subtract your key from the cipher text, and they'll end up with your original message. Coming back to the notation above, this means that
> $P$ is a plain text written in the standard latin alphabet (remember the guidelines from the beginning of this post).
> $C$ is a cipher text, also written in the latin alphabet
> $K$ is a number that specifies the shift (I'll dive into this in a bit)
> $e_k$ is shifting the alphabet by a number of steps defined by $K$
> $d_k$ is shifting the alphabet by a number of steps defined as $-K$ (to invert e_k)

Don't worry, we're almost there. One final problem presents itself. Assume you receive the following cipher text and a key of $\textsf{5}$: $\textsf{HWDUYTLWFUMD}$. Easy enough to do, you convert it into numbers and subtract $\textsf{5}$. But wait, $\textsf{D}$ would be represented by $\textsf{3}$... wouldn't that make the original letter $\textsf{-2}$? This is where the final math part for today comes into play. A method called "modulo":
> The latin alphabet can be represented by the integer modulo $\mod{26}$
> Any number within our modulo ring must be smaller than the modulo value given$^1$. E.g., $50 \mod{26} = 24$, because $50 - 26 = 24$.

$^1$ <sub>Side note: a modulo _can_ still have negative numbers and that might be relevant later on. But in this example, assume a modulo that doesn't have them.</sub>

Basically, imagine a modulo ring like a clock: once you've passed hour number 23 (or 11 in am/pm clocks), you go back to 0 (or 12). The latin alphabet has 26 letters, no more, no less. And therefore, if you need to go before $\textsf{A}$, you go back to $\textsf{Z}$ and continue your journey from there (and the other way around if you need to go beyond $\textsf{Z}$). So our $\textsf{-2}$ becomes...? You guessed it, $\textsf{Y}$. With this little trick, we can reconstruct the original plain text: $\textsf{CRYPTOGRAPHY}$.

Congratulations, you've just successfully constructed your first cryptographic algorithm! I'll spare you the full mathematical notation. But it also illustrates the disadvantage I initially mentioned: if the enemy gets hold of your key, they'll also be able to reconstruct the message (we'll ignore the fact that in this specific case you could just _try_ all 25 possible keys).

So what if we were able to come up with a crypto system that
* can be calculated pretty quick (though not as fast as a symmetrical system)
* makes it (nigh) impossible to infer the plain text from a cipher using the encryption key
* but still allows the people you _want_ to communicate with to decrypt in a reasonable amount of time?

This is where we take a quick look at

#### Asymmetrical crypto systems
These are also known as "public key systems" (a term you might have heard somewhere). The main advantage they have over symmetrical systems is that they allow you to encrypt your communication without ever sharing your decryption key (also called "private key") with anyone. All you need to do is use the namesake public key (which as we'll see is linked to the private key by some pretty interesting math) provided by your conversation partner, and they'll be able to infer the decryption rule from your message and the private key they know. On the downside, the required math makes it a little harder to calculate the keys, and you need bigger/longer keys to reach the same level of security.

We'll explore both concepts in (mathematical) detail in the coming posts. This one has gotten long enough as it is. So we'll slowly wrap it up.

Before I leave, a few questions that you might want to ponder in the meantime:
* Could we find a way to leverage the advantages of both systems while keeping the disadvantages to a minimum?
* Could we use crypto to do more than "just" keep our communication from others?
* How would you securely store a cryptographic key?
* What is more dangerous: a fast computer or a charismatic opponent?