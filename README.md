# Remediator

Remediator is a event bus that handles accepting incoming webhooks and
then executing a desired action based on that payload.

This system is targetted at accepting payloads from monitoring systems
and running remediation steps to avoid involving getting on callers out
of bed for scenarios that can be handled automatically.

### Configuration

It is configured using [HCL][hcl_docs]. It's readable and easily
parsable.

### Examples

Let's imagine we have the following HTTP payload from our monitoring
system.

```json
{
  "id": "d8e8fca2dc0f896fd7cb4cb0031ba249",
  "title": "nginx is not running on web01",
  "details": "web01 has reported that nginx is no longer running",
  "host": "10.0.0.3",
  "tags": [
    "webserver"
  ]
}

```

With the following configuration.

```hcl
strategy {
  conditions [
    {
      payload_type = "json"
      operator     = "contains"
      field_name   = "title"
      field_values = [
        "nginx",
        "not running",
      ]
    }
  ]

  action {
    ssh_exec = "sudo service restart nginx"
  }

  target {
    ec2_instance {
      ipv4 = "${payload.host}"
    }
  }
}
```

**What is happening here?**

Quick version: When our system recieves a JSON payload that has `title`
field and contains the words "nginx" **and** "not running", it will
SSH to the target instance and execute `sudo service restart nginx`

Line by line:

- `strategy` We are defining a new event we want to react to. You can
  have multiple of these per file.
- `conditions` Outline when you want to invoke the `action`.
  - `payload_type` What format does the incoming request look like?
  - `operator` Which comparsion operator to use with the `field_name`
    and `field_value`.
  - `field_name` The value of the key you want to inspect for the
    `field_value`.
  - `field_value` Array of keywords you want to look in the
    `field_name` for.
- `action` What do you want to do if the `conditions` are successful?
  - `ssh_exec` Establish a SSH connection to the host and run the
    command.
- `target` Where should the `action` take place?
  - `ec2_instance` The target is an EC2 instance.
    - `ipv4` Fetch variables from the `payload` and use it to define
      which host the action will take place on.

[hcl_docs]: https://github.com/hashicorp/hcl
