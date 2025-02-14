# Multiple domains configuration

## Separate account namespaces

Given two domains, example.org and example.com. foo@example.org and
foo@example.com are different and completely independent accounts.

All changes needed to make it work is to make sure all domains are specified in
the `$(local_domains)` macro in the main configuration file. Note that you need
to pick one domain as a "primary" for use in auto-generated messages.
```
$(primary_domain) = example.org
$(local_domains) = $(primary_domain) example.com
```

The base configuration is done. You can create accounts using maddyctl using
both domains in the name, send and receive messages and so on.  Do not forget
to configure corresponding SPF, DMARC and MTA-STS records as was
recommended in the [introduction tutorial](tutorials/setting-up.md).

## Single account namespace

You can configure maddy to only use local part of the email
as an account identifier instead of the complete email.

This needs two changes to default configuration:
``` 
storage.imapsql local_mailboxes {
    ...
    delivery_map email_localpart
    auth_normalize precis_casefold
}
```

You also need to make `authorize_sender` check (used in `submission` endpoint)
accept non-email usernames:
```
authorize_sender {
  ...
  auth_normalize precis_casefold
  user_to_email regexp "(.*)" "$1@$(primary_domain)"
}
```
Note that is would work only if clients use only one domain as sender (`$(primary_domain)`).
If you want to allow sending from all domains, you need to remove `authorize_sender` check
altogether since it is not currently supported.

After that you can create accounts without specifying the domain part:
``` 
maddyctl imap-acct create foxcpp
maddyctl creds create foxcpp
```
And authenticate using "foxcpp" in email clients.

Messages for any foxcpp@* address with a domain in `$(local_domains)`
will be delivered to that mailbox.
