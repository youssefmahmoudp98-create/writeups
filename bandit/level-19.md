## Level 18 to 19

Goal: log in as `bandit18`, whose login shell has been modified to immediately disconnect on interactive login, and retrieve the password for level 19.

What was done:

`ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"` -> connected to the server and executed `cat readme` directly as part of the SSH command, bypassing the need for an interactive shell session, after which the password for `bandit19` was returned.

Attempting to log in normally with `ssh bandit18@bandit.labs.overthewire.org -p 2220` resulted in an immediate disconnection right after entering the password. The banner on connection indicated that something in the login process had been changed to cause an instant logout.

This meant that whatever was happening, it was occurring specifically in the interactive shell setup, since the SSH authentication itself succeeded before the disconnect happened.

Rather than relying on an interactive shell, it was recalled that SSH allows a command to be passed directly as part of the connection request, without requiring a shell session to be established. The syntax used was:

`ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"`

This caused the server to execute the given command (`cat readme`) non-interactively and return its output directly, without ever fully starting the interactive shell responsible for the logout. Once the password was authenticated, the command executed, and the output was the password for `bandit19`, all before any interactive shell had the chance to disconnect the session.

What was understood:
- SSH normally opens an interactive shell session after authentication, but it can also be used to execute a single remote command directly.
- Passing a command as part of the SSH invocation causes it to run non-interactively on the remote host, bypassing whatever shell setup would otherwise execute.
- A broken or modified login shell only affects interactive sessions; it does not prevent authentication or command execution triggered outside of that shell.
- Understanding the distinction between authentication and shell initialization is key to working around shell-level restrictions.

Vulnerability type: this level demonstrates a shell restriction bypass. Restricting or modifying a user's interactive shell does not prevent command execution through non-interactive SSH invocations, since authentication and command execution can occur independently of shell initialization.

Real world version: administrators sometimes restrict a user's shell (e.g. setting it to `/bin/false` or a restricted shell) to limit interactive access. However, if SSH command execution is still permitted, this restriction can be bypassed by passing commands directly through the SSH invocation, allowing the user to execute arbitrary commands despite the shell restriction.

Cloud version: similar restricted-access patterns are used in cloud environments, such as locking down service accounts or bastion hosts to prevent interactive shell access. If command execution is not equally restricted, users may still perform actions on the system by passing commands directly, which is why shell restrictions must be paired with proper command and permission controls rather than relied on alone.
