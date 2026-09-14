# Bitvise SSH Client

## Introduction

Bitvise SSH Client is a Windows application for secure remote access to SSH servers and for transferring data through encrypted SSH connections. It supports interactive terminal sessions, SFTP file management, public-key authentication, Kerberos-based single sign-on in Windows domains, SSH port forwarding, dynamic SOCKS tunneling, remote desktop forwarding, and scripted command-line operations.

A connection profile stores the parameters required to access a server, including hostname, SSH port, username, authentication method, terminal settings, and tunneling configuration. The default SSH port is 22, but administrators commonly assign another port when exposing a service to untrusted networks. The client uses SSH protocol version 2 and can maintain the server's host key information so that changes to the server identity can be detected.

Authentication can use passwords or SSH key pairs. The Client Key Manager can generate, import, export, and manage private keys used for authentication. In domain environments, Kerberos authentication can provide single sign-on without repeatedly entering a password. Public-key authentication is particularly useful for administrative access and automated processes because the private key remains under the client's control.

After authentication, the same SSH connection can provide several services. An administrator can open a terminal, transfer configuration files through SFTP, create a tunnel to an internal database, or forward Remote Desktop traffic without exposing the target service directly to the network. This makes the client useful for server administration, secure file exchange, controlled access to private services, and remote troubleshooting while keeping related traffic within the encrypted SSH transport.

## SSH Authentication and Remote Administration

The Login section controls the identity and authentication method used when establishing an SSH session. Password authentication is suitable for environments where passwords are permitted, while public-key authentication is preferable for administrative access and automation. A key pair consists of a private key retained by the client and a public key registered for the corresponding account on the SSH server. The private key can be protected with a passphrase, reducing the impact of unauthorized access to the key file.

The Client Key Manager is the central interface for key-pair operations. It can generate new keys, import existing keys, export keys when required, and associate keys with client authentication. A practical administrative workflow is to create a dedicated key for a specific operational role, install only its public key on the target account, protect the private key with a passphrase, and avoid sharing that key between unrelated administrators or systems.

In Windows domain environments, Bitvise SSH Client can use GSSAPI authentication with Kerberos for single sign-on. When the workstation and server participate in a suitable domain or trusted realm configuration, an already authenticated domain user can establish an SSH session without entering the account password again. The client can also request Kerberos delegation when the environment is configured to permit it, which is relevant when the remote session must access Windows network resources under the user's delegated credentials.

For troubleshooting, distinguish authentication failures from transport failures. If the SSH server cannot be reached, investigate DNS resolution, routing, firewall rules, and the configured SSH port first. If the connection reaches the server but authentication fails, inspect the selected username, key, password, Kerberos state, and server-side account permissions separately.

## SFTP, Terminal Sessions, and File Automation

Bitvise SSH Client provides an integrated terminal and graphical SFTP interface, allowing administrators to combine command execution with file management during the same connection. SFTP transfers files through the SSH transport rather than requiring a separate FTP service or an additional network port. This is useful for deploying configuration files, retrieving logs, exchanging build artifacts, and maintaining application data on remote systems.

A common operational sequence is to connect to a Linux or Windows SSH server, upload a new configuration file through SFTP, open the terminal, verify its permissions and contents, and restart the affected service. For example, an administrator might upload an application configuration, run a syntax-validation command remotely, inspect the service status, and then review the resulting log messages. Keeping these steps in one client reduces context switching during maintenance.

The terminal can be used for interactive shells, remote command execution, and administration tasks supported by the account's shell environment. File-system access should not be assumed to equal administrative access: the permissions available through SFTP or the terminal are determined by the remote account and server configuration.

Bitvise SSH Client also provides command-line components suitable for automation. These can be integrated into deployment or maintenance scripts where an interactive graphical session is undesirable. A typical workflow can authenticate using a managed key, connect to a predefined server, transfer a release package, and execute a remote command to start or validate the deployment. Automated jobs should use dedicated accounts and narrowly scoped credentials rather than personal administrator accounts.

For high-volume or recurring transfers, separate connection profiles for development, staging, and production environments help prevent accidental operations against the wrong host. Profiles should be reviewed periodically so obsolete hosts, credentials, and forwarding rules do not remain available to operators.

## SSH Port Forwarding and SOCKS Tunneling

SSH tunneling allows Bitvise SSH Client to transport TCP connections through an authenticated SSH session. The primary advantage is that an internal service does not need to be exposed directly to the administrator's network. Only the SSH server's listening port needs to be reachable; the tunneled service remains accessible through the encrypted SSH channel.

Local port forwarding is useful when the client needs to access a service reachable from the SSH server. For example, suppose an internal PostgreSQL server listens on `10.10.20.15:5432` and accepts connections only from the SSH host. The client can create a local forwarding rule such as `localhost:15432 -> 10.10.20.15:5432`. A database tool then connects to `localhost:15432`, while the SSH server establishes the connection to the internal database.

Dynamic forwarding provides a SOCKS proxy instead of a fixed destination. Applications configured to use the client's SOCKS proxy can send connections through the SSH server and reach destinations available from that server. This is useful for controlled access to several internal services without creating a separate forwarding rule for every host. Browser traffic can also be routed this way, although the connection between the SSH server and the final destination is not automatically encrypted merely because the traffic traveled through SSH.

Remote forwarding reverses the direction: a service accessible from the client side can be made reachable through a listener associated with the SSH server. This can support controlled administrative scenarios, but forwarding permissions should be restricted on the server whenever possible.

Forwarding is therefore a network-access mechanism, not an authentication bypass. The SSH server still determines whether the account may create tunnels and which destinations or ports are permitted. Keep forwarding rules specific to the required services and remove temporary tunnels after maintenance work is complete.

## Remote Desktop, X11, and Secure Access Workflows

Bitvise SSH Client can use SSH forwarding to provide access to graphical services that should not be exposed directly. A practical example is Remote Desktop forwarding: after establishing the SSH connection, the client can create the required forwarding and launch the Windows Remote Desktop client against the forwarded endpoint. The RDP service can remain inaccessible from the public network while administrative access is provided through SSH.

This architecture adds an important security boundary. The SSH server's host key authenticates the remote SSH endpoint before the forwarded RDP connection is established. Network firewalls can therefore allow SSH access while blocking direct inbound RDP traffic. Public-key authentication can further reduce exposure to password guessing, provided the SSH account and key are properly managed.

X11 forwarding addresses a different requirement. Instead of forwarding an entire remote desktop, it allows compatible graphical applications running on the remote system to display their windows through the SSH connection. This can be useful for Unix administration tools that require a graphical interface but do not justify a complete desktop session. X11 forwarding should be enabled only for accounts that require it because graphical forwarding increases the capabilities available through an SSH session.

Bitvise SSH Client can also be used as an FTP-to-SFTP bridge, allowing software that expects an FTP-style interface to exchange files through SFTP. This is valuable when an existing application cannot natively speak SFTP but must communicate with a secure file-transfer endpoint.

In operational environments, these capabilities should be managed as controlled access mechanisms. Use dedicated connection profiles for distinct administrative roles, limit SSH access to only the services required for each task, and disable persistent tunnels when they are not actively needed for temporary maintenance activities.
