---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Visual Cryptography"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2021-10-03T19:50:50+02:00
lastmod: 2021-10-03T19:50:50+02:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

For the Fêtes de la Science in my university, I had to work a bit on visual cryptography and I thought it could be a nice blog post.

{{< toc >}}

__Disclaimer__ : In this article we are going to use modulus. If you don't know what it is, I have written an a small appendix on this at the end.

## Cryptography



## One Time Pad

In the area of cryptography, there is a scheme that is known to be unconditionnaly secure, but ressource expensive, buceause it requires a key that is as long as the original message. It is called the One Time Pad (OTP).

Let's have a first example with letters. Let's suppose we exchange message only composed of the uppercase letters A to Z. Each letter can be mapped uniquely to an integer between 0 and 25 (A is 0, B is 1, ..., Z is 25). Now let's suppose that we want to exchange the message 

<center>
CRYPTOGRAPHY
</center>

The OTP requires that we have a key with the same length as the message. I hence generate a random string of length 13 with uppercase letter and I have the following key

<center>
CRAZHSOFTMFN
</center>

To have the encrypted message, we will have to make a sum between the message and the key. In fact for each character, we are going to make the following operations : 

1. Get the indexes corresponding to the char of the message and the key between 0 and 25;
2. Sum the two indexes with modulus 26 (that ensures that the result stays between 0 and 25);
3. We now have the index of the encrypted string.

Let's do it witt the example : 

| Message | Key | Index message | Index key | Sum | Sum modulus 26 | Encrypted character |
| ------- | --- | ------------- | --------- | --- | -------------- | ------------------- |
| C       | C   | 2             | 2         | 4   | 4              | E                   |
| R       | R   | 17            | 17        | 34  | 8              | I                   |
| Y       | A   | 24            | 0         | 24  | 24             | Y                   |
| P       | Z   | 15            | 25        | 40  | 14             | O                   |
| T       | H   | 19            | 7         | 26  | 0              | A                   |
| O       | S   | 14            | 18        | 32  | 6              | G                   |
| G       | O   | 6             | 14        | 20  | 20             | U                   |
| R       | F   | 17            | 5         | 22  | 22             | W                   |
| A       | T   | 0             | 19        | 19  | 19             | T                   |
| P       | M   | 15            | 12        | 27  | 1              | B                   |
| H       | F   | 16            | 5         | 12  | 12             | M                   |
| Y       | N   | 24            | 13        | 37  | 11             | L                   |

and hence the encrypted message is then 

<center>
EIYOAGUWTBML
</center>

Now we are going to get the decryption process and then see why it's properly secure.

The decryption process is similar to the encryption process :

1. Get the indexes corresponding to the char of the encrypted message and the key between 0 and 25;
2. Compute the difference between the index of the encrypted message and the index of the key modulus 26;
3. We now have the index of the message.



## Visual Cryptography

### 2-shares

### n-shares

### Known shares

## Appenxix A : Modular arithmetic
