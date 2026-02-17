# `bw2ssh` - Generate `.ssh` from Bitwarden

A single CLI command to automatically create your `.ssh/config` file and import necessary public keys based on custom fields from your Bitwarden vault.

Vaultwarden works out of the box, for sure.

## Motivation

I use [Vaultwarden](https://github.com/dani-garcia/vaultwarden) as password manager and I store my SSH keys with it. To use them, I have to use the [Bitwarden SSH agent](https://bitwarden.com/help/ssh-agent/#configure-bitwarden-ssh-agent) and most of the time, people are hit with a limitation : when you're asked for an SSH key, it tries each of your keys until finding the right one.

Sadly, most of the time there's a max authentication limit that you'll always hit if you have more than 5 keys stored in there.

This is a [known](https://github.com/bitwarden/clients/issues/13401) [issue](https://community.bitwarden.com/t/ssh-agent-allow-specifying-limiting-private-key-offers) and the only solution currently is to create an SSH configuration `.ssh/config` where you add all your hosts and point them to the public key.

```config
Host remote
  IdentitiesOnly yes
  IdentityFile ~/.ssh/remote.pub
```

You have to manually do this process and you may find it exhausting to do and redo this process on all your devices if you have dozens of keys that you use daily.

## Installation

You can use [`cargo`](https://doc.rust-lang.org/cargo/) to install the CLI globally.

```sh
cargo install --git https://seed.vexcited.com/z3R6nHv6KaM8MpVuXPwbttyCcc4Zo.git
```

You also need the [Bitwarden CLI](https://bitwarden.com/help/cli/#download-and-install) since we're using it to pull the SSH keys and custom fields.

```sh
npm install --global @bitwarden/cli
```

## Variables

You'll have to add custom fields detailled below in the SSH keys you want to handle. You have to respect the uppercase casing.

### `HOSTS` (required)

A comma-separated value containing all the hosts for the corresponding key.

Maps to the `Host` key.

#### Example

Defining <kbd>HOSTS</kbd> to `192.168.1.10,rpi` will generate the following.

```
Host 192.168.1.10 rpi
  IdentitiesOnly yes
  IdentityFile ~/.ssh/bw_<hash>.pub
```

### `HOSTNAME` (optional)

If you have multiple hosts and that you use some of them as aliases, you might
want to setup a specific hostname.

Maps to the `HostName` key.

#### Example

Defining <kbd>HOSTNAME</kbd> to `192.168.1.10` could generate the following.

```
Host 192.168.1.10 rpi
  HostName 192.168.1.10
  IdentitiesOnly yes
  IdentityFile ~/.ssh/bw_<hash>.pub
```

### `USER` (optional)

Specifies the user to log in as. This can be useful when a different user name is used on different machines. This saves the trouble of having to remember to give the user name on the command line.

Maps to the `User` key.

#### Example

Defining <kbd>USER</kbd> to `dietpi` could generate the following.

```
Host 192.168.1.10 rpi
  HostName 192.168.1.10
  User dietpi
  IdentitiesOnly yes
  IdentityFile ~/.ssh/bw_<hash>.pub
```

## Usage

> You must know that running the CLI will overwrite your current `~/.ssh/config` file, don't forget to do a backup before-hand if you're unsure about your configuration yet using `cp ~/.ssh/config ~/.ssh/config.bak`.

You have to get a session token using the following commands.

```sh
# don't forget to update the server if you're self-hosting.
bw config server https://yourvault.example.com

# check the documentation @ https://bitwarden.com/help/cli/#log-in
bw login

# if you're already authenticated, unlock the vault to get a new token.
bw unlock

# also if you've updated your ssh keys, don't forget to sync the CLI.
bw sync
```

Now you can run this tool with the token as argument.

```sh
bw2ssh --token cj[redacted]MA==
```

You're done! You can now check your `~/.ssh` directory, everything should be there and properly configured.
