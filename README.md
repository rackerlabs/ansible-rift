Overview
--------

This is an ansible play to deploy [CloudRift](https://github.com/CloudRift/Rift) on a Rackspace cloud server. It will:

1. Create a Rackspace cloud server
2. Start CloudRift running on `localhost:8000` on that server
3. Configure nginx with basic auth on port 80, proxying to Rift

The play is configurable via environment variables so it is easily usable from a Jenkins job.

Installing
----------

Everything you need to install should be in the `requirements.txt` (and you should create a vritualenv to work in first).

    pip install -r requirements.txt

Deploying CloudRift
-------------------

To run the playbook do the following:

    $ cp configure-ansible.sh.sample configure-ansible.sh
    
    # BE SURE TO EDIT configure-ansible.sh TO FILL IN EMPTY VARIABLES
    
    $ source configure-ansible.sh
    $ ansible-playbook deploy-rift.yml
    
    # grab the ip address of our new server
    # you may need to `apt-get install jq`, or use grep/sed instead
    $ RIFT_IP=`./inventory/rax.py --host "${CLOUDRIFT_SERVER_NAME}1" | jq -r .rax_accessipv4`
    
    # make requests to rift
    $ curl $CLOUDRIFT_USERNAME:$CLOUDRIFT_PASSWORD@$RIFT_IP
    
The above has all the steps you need. More detail is below:

1. Configure the play by setting environment variables:
    * Create a file with these variables in it: `cp configure-ansible.sh.sample configure-ansible.sh`
    * Edit `configure-ansible.sh` to set the empty environment variables
    * Run `source configure-ansible.sh` to set those environment variables
2. Run `ansible-playbook deploy-rift.yml`
    * NOTE: If your `RACKSPACE_SSHKEY_PUB` is different than `~/.ssh/id_rsa.pub`, then you will need to tell ansible where the corresponding private key is with the `--private-key` flag. You should run: `ansible-playbook deploy-rift.sh --private-key=/path/to/id_rsa`
3. Grab the ip address of your server. The `./inventory/rax.py` script grabs info about your Rackspace cloud servers:
    * To print the adress of your CloudRift server: `./inventory/rax.py --host "${CLOUDRIFT_SERVER_NAME}1" | grep rax_accessipv4`. You can then grab the ip manually.
    * **Better**: If you have [jq](http://stedolan.github.io/jq/) installed (just run `apt-get install jq`), you can easily filter json values. Run `RIFT_IP=$(./inventory/rax.py --host "${CLOUDRIFT_SERVER_NAME}1" | jq -r .rax_accessipv4)`
4. Check that you can make api requests to CloudRift. Remember that you need the username and password for this: `curl $CLOUDRIFT_USERNAME:$CLOUDRIFT_PASSWORD@$RIFT_IP`

Cleaning up
-----------

To tear down your server with ansible:

    source configure-ansible.sh
    ansible-playbook cleanup-rift.yml
