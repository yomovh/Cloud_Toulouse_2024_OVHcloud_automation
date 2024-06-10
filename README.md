# Notes for the demo that illustrates the talk "How to industrialize the deployment of your solutions with OVHcloud ?" given at OVHCloud Toulouse Meetup

## OVHcloud control panel interface

Copy public key
```bash
cat ~/.ssh/id_rsa.pub |pbcopy	
```
Add [cloud-init script](cloud-init.yaml)

The cloud-init log are available in `/var/log/cloud-init-output.log`


## API console 

[api.ovh.com](https://api.ovh.com)

Search for Public Cloud Compute tile or go directly to the [instance page](https://eu.api.ovh.com/console/?section=%2Fcloud&branch=v1#get-/cloud/project/-serviceName-/instance)

## Python API 

[Documentation first steps with API](https://help.ovhcloud.com/csm/en-gb-api-getting-started-ovhcloud-api?id=kb_article_view&sysparm_article=KB0042784) or in [French](https://help.ovhcloud.com/csm/fr-api-getting-started-ovhcloud-api?id=kb_article_view&sysparm_article=KB0042789) provides link to [token creation page](https://api.ovh.com/createToken/?GET=/*&POST=/*&PUT=/*&DELETE=/*)

Fill in the IP restrictions if needed.

The following environment variables needs to be filled or put in an `.ovh.conf` file 

```
[default]
; general configuration: default endpoint
endpoint=ovh-eu

[ovh-eu]
; configuration specific to 'ovh-eu' endpoint
application_key=
application_secret=
consumer_key=
```

Or in environment variable (required for the last example because of acme provider in terraform)
```bash
export OVH_ENDPOINT=ovh-eu
export OVH_APPLICATION_KEY=
export OVH_APPLICATION_SECRET=
export OVH_CONSUMER_KEY=
```

Install the python OVHCloud wrapper ([github page](https://github.com/ovh/python-ovh))
```bash
python3 -m venv /home/ubuntu/venv
source venv/bin/activate
pip install ovh 
```

Now go back to your API console browser tab, copy the Python code and paste a 
```bash
vim list_instances.py
```
* Remove the authentication in the code (we are using configuration file)
* Replace the {servicename} by the value copied from the control panel (below you project name)

```python
import json
import ovh
client = ovh.Client()

result = client.get("/cloud/project/{serviceName}/instance")

# Pretty print
print(json.dumps(result, indent=4))
```

Run !
```sh
python list_instances.py |jq
```
 
## Openstack CLI 

From Manager, open the *Horizon* link, Identity > Application Credentials > Create Application Credentials 
scp the file content on the environment, source it. 

```bash
pip install python-openstackclient
openstack server list
```


## Terraform / opentofu

Example from this [project](https://github.com/yomovh/tf-at-ovhcloud)

```bash 
git clone https://github.com/yomovh/tf-at-ovhcloud
cd tf-at-ovhcloud/simple_http_lb_with_prom_grafana
```

Add the variable to a file `variables.tfvars`
```
dns_zone                          = "galsandbox.ovh"
acme_email_address                = ""
instance_nb = 4
ovh_public_cloud_project_id = “”
```


Init the providers
```bash
tofu init
```
Launch code
```bash
tofu apply -var-file=variables.tfvars
```

Get avnadmin password from MacOS
```bash
ssh ubuntu@IP 'cd tf-at-ovhcloud/simple_http_lb_with_prom_grafana && tofu output -raw grafana_password'|pbcopy
```
