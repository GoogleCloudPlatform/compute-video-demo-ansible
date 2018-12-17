## compute-video-demo-ansible

This is the supporting documentation for
<a href='https://www.youtube.com/watch?v=FF-HfP_OHpU'>Using Ansible with Google</a>
video.

The goal of this repository is to provide the extra detail necessary for
you to completely replicate the recorded demo. The video's main goal
is to show a quick, fully working demo without bogging you down with all
of the required details so you can easily see the "Good Stuff".

So for interested viewers wanting to replicate the demo on their own, this
repository contains all those necessary details.

## Google Cloud Platform Project

1. You will need to create a Google Cloud Platform Project as a first step.
Make sure you are logged in to your Google Account (gmail, Google+, etc) and
point your browser to https://console.cloud.google.com/projectselector/compute/instances. You should see a
page asking you to create your first Project.

1. When creating a Project, you will see a pop-up dialog box. You can specify
custom names but the *Project ID* is globally unique across all Google Cloud
Platform customers.

1. It's OK to create a Project first, but you will need to set up billing
before you can create any virtual machines with Compute Engine. Find the menu icon at the top left, 
then look for the *Billing* link in the navigation bar.

1. In order for `ansible` to create Compute Engine instances, you'll need a
[Service Account](https://cloud.google.com/compute/docs/access/service-accounts#serviceaccount). 
It's recommended that you create a new Service Account (don't use the default), called 'demo-ansible', for this demo.
Make sure to create a new JSON formatted private key file for this Service Account. Also, note the *Email address* 
of this Service Account (should be `demo-ansible@YOUR_PROJECT_ID.iam.gserviceaccount.com`) since 
this will be required in the Ansible configuration files.

1. Next you will want to install the
[Cloud SDK](https://cloud.google.com/sdk/) and make sure you've
successfully authenticated and set your default project as instructed.

1. You will also need to make sure and set up SSH keys that will allow you to
access your Compute Engine instances. You can either
[manually generate the keys](https://cloud.google.com/compute/docs/instances/adding-removing-ssh-keys#createsshkeys) and
paste the public key into the [metadata server](https://console.cloud.google.com/compute/metadata/sshKeys)
or you can use `gcloud compute ssh` to access an existing Compute Engine instance
and it will handle generating the keys and uploading them to the metadata
server. For this demo, it is assumed you have opted to use
`gcloud compute ssh` and your private key is located at `$HOME/.ssh/google_compute_engine`.

## Software

1. Install Dependencies.  On Debian-7, you may run the following to install them.

    ```
    sudo apt-get install -y build-essential git python-dev python-pip
    ```
1. Install Ansible with the
[running from source instructions](http://docs.ansible.com/intro_installation.html#running-from-source).

1. Install GCP module dependencies

    ```
    pip install googleauth requests
    ```

1. For the purposes of the demo, you can set a couple of environment variables
to simplify your commands and SSH interactions.

    ```
    export ANSIBLE_HOST_KEY_CHECKING=False
    ```

## Ansible demo setup

1. Check out this repository so that you can use pre-canned configuration
and demo files.
    ```
    cd $HOME
    git clone https://github.com/GoogleCloudPlatform/compute-video-demo-ansible
    ```

1. Edit the `gce_vars/auth` file and specify your Project ID in the
`project_id` variable, Service Account email address in the `service_account_email` variable,
and the location of your JSON key (downloaded earlier) in the `credentials_file` variable.
    ```
    ---
    # Google Compute Engine required authentication global variables
    # (Replace 'YOUR_PROJECT_ID' with the Project ID used in creating your GCP project.)
    project: YOUR_PROJECT_ID
    service_account_file: /path/to/your/json_key_file
    ```

# Demo time!

You've now completed all of the necessary setup to replicate the demo as
shown on the video. Now, you'll use `ansible-playbook` to create and bootstrap
the instances, install Apache, and set up a Compute Engine load-balancer.

## Run the playbook

Use the `ansible-playbook` command to create the instances based on the
attributes in the configuration files. For the four instances, this
should take roughly 2 minutes to create the new Compute Engine
instances

```
ansible-playbook -i ansible_hosts site.yml
```

1. The output from this command will display the public IP address associated
with your new load-balancer. You can also look in the Developers Console
under *Networking &gt; Load balancing* and find the Forwarding Rules under the 
"Advanced" menu.

1. Ok, let's test it out! Put the public IP address of your load-balancer into
your browser and take a look at the result. Within a few seconds you should
start to see a flicker of pages that will randomly bounce across each of your
instances.

    For the demo, a javascript function is set to fire when the page loads
that pauses for a half-second, and then reloads itself. Since we installed
a modified Apache configuration file to disable client-side caching *and* we
enabled Apache's `mod_headers`, each "reload" results in a new HTTP request
to the page. This is just a fancy hands-free way of asking you to do a
"hard refresh" of the load-balancer IP address in order to see the cycling
between instances.

# All done!

That's it for the demo. There is a lot of other functionality for
Compute Engine in Ansible. Please take a look at the `gce*`
[modules](http://docs.ansible.com/ansible/list_of_cloud_modules.html#google) for a full set
of modules and instructions.

## Cleaning up

When you're done with the demo, make sure to tear down all of your
instances and clean-up. You will get charged for this usage and you will
accumulate additional charges if you do not remove these resources.

Fortunately, you can use the `clean-up.yml` playbook for destroying these
demo Compute Engine resources. The following command can be used to destroy
all of the resources created for this demo.

```
ansible-playbook clean-up.yml
```

## Troubleshooting

* Make sure your GCP Project Name is set.  To set it, go to *API Manager &gt; Credentials &gt; OAuth consent screen*.

* If Ansible keeps saying that requests or googleauth aren't installed, run `which python` to see which copy of the
  Python interpreter you installed the dependencies to. In ansible_hosts, append `ansible_python_interpreter=PATH_FROM_WHICH_PYTHON`
  to the `localhost` line


## Contributing

Have a patch that will benefit this project? Awesome! Follow these steps to have it accepted.

1. Please sign our [Contributor License Agreement](CONTRIB.md).
1. Fork this Git repository and make your changes.
1. Run the unit tests. (gcimagebundle only)
1. Create a Pull Request
1. Incorporate review feedback to your changes.
1. Accepted!

## License
All files in this repository are under the
[Apache License, Version 2.0](LICENSE) unless noted otherwise.

