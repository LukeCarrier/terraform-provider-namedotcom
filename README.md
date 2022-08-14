# Terraform Provider for [name.com](https://name.com)

Special thanks to @mhumeSF for the original [name.com provider](https://github.com/mhumeSF/terraform-provider-namedotcom).

[API Docs](https://www.name.com/api-docs)

Supported features:

- Set DNS records
- Set NS records

## Usage

### How to install the provider

```HCL
terraform {
  required_providers {
    namedotcom = {
      source  = "lexfrei/namedotcom"
      version = "1.1.6"
    }
  }
}
```

### How to create the provider

Username and token must be generated from your account, [here](https://www.name.com/account/settings/api).

```HCL
provider "namedotcom" {
  username = var.namedotcom_username
  token    = var.namedotcom_token
}
```

### Example usage

```HCL
// example.com CNAME -> bar.com

resource "namedotcom_record" "bar" {
  domain_name = "example.com"
  host = ""
  record_type = "cname"
  answer = "bar.com"
}

// foo.example.com -> 10.1.2.3

resource "namedotcom_record" "foo" {
  domain_name = "example.com"
  host = "foo"
  record_type = "A"
  answer = "10.1.2.3"
}
```

Many records per domain example

```HCL
resource "namedotcom_record" "domain-me" {
  domain_name = "domain.me"
  record_type = "A"
  for_each = {
    "" = local.t6
    www = local.t8
    www1 = local.t8
    www2 = local.t9
  }

  host = each.key
  answer = each.value
}
```

Setting nameservers from a generated hosted_zone

```HCL
provider "aws" {
  region = "us-west-2"
}

provider "namedotcom" {
  token = "..."
  username = "..."
}

resource "aws_route53_zone" "example_com" {
  name = "example.com"
}

resource "namedotcom_domain_nameservers" "example_com" {
  domain_name = "example.com"
  nameservers = [
    "${aws_route53_zone.example_com.name_servers.0}",
    "${aws_route53_zone.example_com.name_servers.1}",
    "${aws_route53_zone.example_com.name_servers.2}",
    "${aws_route53_zone.example_com.name_servers.3}",
  ]
}
```

### How to import record

You need to use format "domain:id" as last parameter for import command

```bash
# Import single record
terraform import namedotcom_record.example_record domain_name:recordId

# Import one of mentionned records in for_each
terraform import 'namedotcom_record.example_record["hostname"]' domain_name:recordId
```

To get recordId, you need to use namedotcom API for domain ListRecords and use ID for appropriate host

```bash
curl -u 'username:token' 'https://api.name.com/v4/domains/example.org/records'
```
