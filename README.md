## Nagios Fuze Notifications

This simply takes Nagios environment variables ([see here](https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/3/en/macrolist.html)), and builds a quick text block to send to a Fuze webhook -   ([see here](https://help.fuze.com/hc/en-us/articles/360036848673))

Create two new commands in your `commands.cfg`:

```
define command {
  command_name notify-host-by-fuze
  command_line /usr/bin/printf "%b" "*Host: $HOSTNAME$*\nNotification Type: $NOTIFICATIONTYPE$\n*State: $HOSTSTATE$*\nAddress: $HOSTADDRESS$\n*Info: $HOSTOUTPUT$*\nDate/Time: $LONGDATETIME$\nHost Info URL: $HOSTINFOURL$\n" | curl -H 'Content-Type: text/plain' https://api.fuze.com/chat/v2/webhooks/incoming/token --data-binary @-
}

define command {
  command_name notify-service-by-fuze
  command_line /usr/bin/printf "%b" "*Host: $HOSTALIAS$*\nNotification Type: $NOTIFICATIONTYPE$\nService: $SERVICEDESC$\nAddress: $HOSTADDRESS$\n*State: $SERVICESTATE$*\n\nDate/Time: $LONGDATETIME$\nService Info URL: $SERVICEINFOURL$ \nAdditional Info:\n*$SERVICEOUTPUT$*\n" | curl -H 'Content-Type: text/plain' https://api.fuze.com/chat/v2/webhooks/incoming/token --data-binary @-
}
```

Also ensure there's a separate "fuze" contact, or similar in your `contacts.cfg`:

```
define contact {
  contact_name fuze
  use fuze-contact
  alias fuze Contact
  email some@example.com
}
```

Update your `contactgroup` as well:

```
define contactgroup {
  contactgroup_name everyone
  alias Everyone
  members nagiosadmin,fuze
}
```

Note, if you use contact groups extensively, you could also add that to the script, and then alert groups/roles with `@mentions` in your alerts.

The last thing to worry about adding, is your base contact, likely in your `templates.cfg`:

```
define contact {
  name fuze-contact
  service_notification_period 24x7
  host_notification_period 24x7
  service_notification_options w,u,c,r,f,s
  host_notification_options d,u,r,f,s
  service_notification_commands notify-service-by-fuze
  host_notification_commands notify-host-by-fuze
  register 0
}
```
