---
layout: post
toc: true
title: Shell Script to Create Passwords as Required by Locaweb Hosting Services
permalink: shell/shell-script-generate-passwords-locaweb.html
date: 2018-05-06 09:37:00
excerpt: Locaweb hosting services advise that passwords have special chars, but it only allows a specific set of them, and this script generates passwords that conform to their requirements.
categories: shell, linux-gamming
---

# Introduction

I host some websites and applications on Locaweb. When creating databases or email accounts, they only allow certain special characters to be included. The problem is that you first have to submit the password before they show you which special characters are allowed (a very annoying behaviour; they should show that before letting the user attempt to submit the form).

I tried using some tools to generate the password, like `openssl rand …​` and `pwgen`, but I could not bend those to my will of creating passwords following the constraints required, so I decided to make something myself.

> **Note**
>
> I do not claim my script/approach is secure, or smart, or whatever. It just worked for my needs.

# The Script

**pwlw.sh.**

``` bash
#!/bin/bash

# Author: Fernando Basso
# Email: fernandobasso.br@gmail.com
# Tested to work on bash >= 3.2 but it probably works
# on version 3.0 and 3.1 as well.
# No external dependencies are required besides `cat'.

# Min length of passwords.
min=4

# Max length of passwords.
max=48

# Default length of generated passwords.
pwlen=11

#
# Display usage help.
# @param number: exit status
#
function usage () {
    # Default exit status is zero.
    exit_status=0

    if [[ $1 ]] ; then
        exit_status=$1
    fi

cat << EOF
USAGE: ${0##*/} [LEN]

Generates a password according to Locaweb's guidelines.
Especially, they allow only a specific set of special
characters, and this script makes use of only the
allowed characters.

OPTIONS

  --help  Show this help and exit.
  LEN a number indicating the length of the password, default is 11

EXAMPLES

Generate a password with 11 characters:

    ${0##*/}

Generate a password with 8 characters:

    ${0##*/} 8

EOF
    exit $exit_status
}

# If we have the paramter --help or a number.
if [[ $1 ]] ; then
    # If it is indeed a number.
    if [[ $1 =~ ^[0-9]+$ ]] ; then
        # It can't be too small or too large.
        if [[ $1 -lt 4 || $1 -gt 48 ]] ; then
            printf '\n!!! Password must be between %d and %d characters long!!!\n\n' $min $max 1>&2
            usage 1
        else
            pwlen=$1
        fi
    elif [[ $1 == --help ]] ; then
        # If they asked for help, exit status is zero.
        usage 0
    else
        # Some incorrect ivokation of the script. Exit status 1.
        usage 1
    fi
fi

# Locaweb admin panels accepts these, and only these special chars.
locaweb_chars='-@!*_:=#/{}[]'

nums=0123456789
lower=abcdefghigklmnopqrstuwxz
upper=ABCDEFGHIGKLMNOPQRSTUWXZ
lwchrs='-@!*_:=#/{}[]'

# Using $lwchars twice so special chars have a bit
# more chances of being picked.
allowed_chars="${lwchrs}${nums}${lower}${upper}${lwchrs}"

# Get random number from 0 to length of `lower'.
i=$(( (RANDOM % ${#lower}) ))

# Locaweb passwords require first character to be a letter.
# We have just assumed a lowercase first letter is good enough.
first_char=${lower:i:1}

pw=$first_char
while [ ${#pw} -lt $pwlen ] ; do
    # i is a random number between zero and length of allowed_chars.
    i=$(( (RANDOM % ${#allowed_chars}) ))
    pw+="${allowed_chars:i:1}"
done

printf 'Generated a %d-character long password:\n' "${#pw}"
printf '\n\t%s\n' $pw


```

[https://gitlab.com/fernandobasso/dotfiles/blob/master/bin/pwlw.sh](https://gitlab.com/fernandobasso/dotfiles/blob/master/bin/pwlw.sh){:target="_blank"}

# Using pwlw.sh

Nothing much to say, but you can show the help:

    pwlw.sh --help

Generate a password with 11 chars (default):

    pwlw.sh

Generate a password with 14 chars:

    pwlw.sh 14

# Screenshots

![displaying help]({{ site.base_url }}/imgs/shell/pwlw-script-help.jpg)

![generating passwords]({{ site.base_url }}/imgs/shell/pwlw-script-generating-passwords.jpg)
