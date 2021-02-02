---
subtitle: "[]{#_top .anchor}**[I Did a Deep Dive on Upgrading To
  Terraform v0.14 and This is What I Found]{.ul}**"
---

I first became curious on the features of terraforms latest major
release v0.14, as it is my first upgrade. I had a lot of questions going
into, questions like, "Can I upgrade directly from the CLI?", "Do I have
to have to reset the path for the new binaries?", and "How long will
Terraform continue to offer support for my current version?" After
reading Terraforms documentation, I felt I had even more. If you share
these questions, don't worry, this articles aim is to address them. I
find when I'm having difficulty understanding things, the best approach
is to try them out. After trying them out I thought I couldn't be the
only person with these questions, and so it seemed like a good idea to
share what I found. If you've upgraded with Terraform before, rest
assure, the steps are the same as in all previous version. Below is a
set of links to help you navigate the document to a specific section.
Also throughout the article are links to Terraforms documentation to aid
in your upgrade process.

[Upgrade Determination](#Upgrade_Determination)

[Upgrade Paths](#Upgrade_Paths)

[Sensitive Parameter Explored](#Sensitive_Parameter)

[Download and Installing]{.ul}:

[Upgrading](#Upgrading)

[Windows GUI](#Windows_GUI)

[Mac OS/Linux](#MAC_OS_Linux)

[Dependency Lock File](#Dependency_Lock_File)

[Upgrade Determination](\l)

Performing version updates and upgrades can be daunting, and require
lots of analysis. This is especially true in mid to large size
enterprises with multiple environments or Workspaces (what Terraform
calls environments) where there are often many configurations.
Therefore, it's imperative that all the question that can be answered
prior to the upgrade are. Even in a well-planned rollout there may still
be unforeseen downtime or other delays. Upgrading to v0.14 series In
Terraform is no different and if not thoroughly investigated could have
you running more terraform plan and apply commands then necessary to
find the errors, or rereading lengthy documentation. There are many
factors to consider prior to upgrading; factors such as changes to
configurations, loss of some functionality in exchange of newer or
updated features, and the amount of effort involved when migrating from
one version to the next. Included in these scenarios is how to upgrade.
Questions such as organizational size, configuration complexity, and how
many versions your team intends to upgrade through? This leads us to
some important changes introduced in this latest major release. For
instance, the expansion of features such as the [sensitive
parameter](#Sensitive_Parameter) and brand-new features like the
[dependency-lock-file](#Dependency_Lock_File). However, the scope of
this blog covers the upgrade process and only touches on some of the
newer features in Terraform v0.14 briefly. For further information on
the features that could throw an error and force you to change
configurations visit Terraforms
[changelog](https://github.com/hashicorp/terraform/blob/v0.14/CHANGELOG.md)
page at Github or this handy blog at
[ITNEXT](https://itnext.io/upgrading-to-terraform-0-14-experience-warning-18ea3f4bc396)
for more information.

[]{#Upgrade_Paths .anchor}

[Upgrade Paths](#_top)

Terraforms recommended upgrade path consist of upgrading through all
minor releases (a version within a version which contains fixes) before
upgrading fully to the next version. This means starting at the lowest
minor release and ending at the highest, thereby applying all the bug
fixes for that version, and any updates to the provider plugins. If you
intend to upgrade through multiple versions, best practice would warn
against drastic changes. Mainly because of the changes from one version
to the next requires omitting items from a configuration, and future
deprecation of some features like versions from the provider block. In
short, writing a new plan to meet the version specification, only to
rewrite the plan yet again on the next upgrade. In such cases an
incremental approach would be best, as Terraform as no plans to sunset
any of their previous versions so you should not feel as though you need
to bring your version current immediately. Smaller and less complex
environments allow for more maneuverability however, and Terraform only
recommends these but does not enforce them. Because of this, another
option is to skip a version and rewrite your plan once you've landed at
the desired version. Be warned though about potential problems showing
up later, as performing an upgrade in this manner could add to the
complexity of configuration changes to your environment, and increase
the unknowns. However, if you are only running in a lab, and referencing
the local backend, this method should be fine. As an example, in the
pictures below, I upgraded from v0.13.5 -- 0.14.4.

![](media/image1.PNG){width="6.469653324584427in"
height="2.135715223097113in"}

When I first started working with Terraform, I simply grabbed the most
recent download and never paid any attention to the previous versions,
as logic tells me the binaries contain all previous updates. However,
complexity grows over time after provisioning infrastructure and writing
configuration, therefore more care needs to be taken. To get started on
the upgrade process and to download older releases you can go to
<https://releases.hashicorp.com/terraform/>.

[]{#Sensitive_Parameter .anchor}

[Sensitive Parameter Explored](#_top)

The Sensitive parameter is more extensive in v0.14, expanding its
behavior to include variables as well. This means that when an
expression references the variable which contains the sensitive
parameter, the sensitivity value is carried over in order to suppress a
value in the CLI output (not the TF state file, which is the default
behavior carried over from previous versions). Below is an example
configuration I grabbed from
<https://github.com/hashicorp/terraform/pull/26183> with also the CLI
output included at the bottom.

terraform {

experiments = \[sensitive_variables\]

}

variable \"foo\" {

sensitive = true

}

resource \"some_resource\" \"bar\" {

some_val = var.foo

}

CLI Output:

\# some_resource.bar will be created

\+ resource \"some_resource\" \"bar\" {

\+ id = (known after apply)

\+ some_val = (sensitive)

}

Several issues existed within the earlier minor releases, but have since
been fixed in v0.14.4. One bug was found when used with data sources (a
way for terraform to use data elsewhere in either your local or remote
configuration). To avoid problems, any upgrade should be to v0.14.4 and
beyond. To view all the bug fixes relating to the sensitive value click
[here](https://github.com/hashicorp/terraform/blob/v0.14/CHANGELOG.md).

[Upgrading](\l)

I was disappointed to find there is no magic terraform command or set of
commands that will grab the binaries of the most recent minor release
while also taking a snapshot of your current configuration. What does
that mean? It means your kinda stuck performing these actions on your
own, similar to an initial download and install. In addition, the
command terraform 0.13upgrade is no longer supported in v0.14, and
neither is the state snapshot feature. However, it is recommended
terraform 0.13upgrade be ran prior to upgrading (records the source
address and providers in use), and a following a few steps to manually
take a state snapshot. In order to manually make a snapshot of your
state Terraform recommends performing actions at the CLI prior to
install. If you've worked with terraform in the past this shouldn't be
too hard. just as in all previous major releases, the steps are to run
terraform init, plan, and apply at the CLI and upgrading though all
minor releases. Performing these steps allows you to migrate your
current settings and infrastructure to the newer version (a lift and
shift).

The following should be performed in the previous version of Terraform
(0.13.6) before upgrading:

-   Terraform init: initializes the working directory, and a refresh
    also takes place when the init command is ran, which matches the
    state file to the actual infrastructure, more information can be
    found at
    [ttps://www.terraform.io/docs/commands/refresh.html](https://www.terraform.io/docs/commands/refresh.html%20%20)
    .

-   Terraform plan: ensures there are no pending changes to the state
    file, and that the command can be ran. Pending changes can "add
    additional unknowns", as said by Terraform. Basically it's best to
    make sure your infrastructure and state file are the way you want
    them.

-   Terraform apply: This is the step that creates a snapshot of your
    infrastructure prior to upgrading. But Terraform no longer offers
    the state snapshot feature, meaning the terraform apply command must
    be ran at least once so it can complete its state format upgrade.

[Windows GUI](\l)

Since we're only upgrading and not downloading, we won't have to change
or update the path for the binaries. I've included some examples below
to illustrate the process. navigate to the downloads page and grab the
binaries, find the location in your local file system where the current
binaries are kept and replace, move the previously used binaries to an
alternate location, and run. For my example, I had created a folder
named "Binaries" that held the v0.13 download that existed in the root
of C volume, inflated the zip file, and replaced the old binaries.

1). Navigate to the [downloads
page](https://www.terraform.io/downloads.html) and grab either the 32-
or 64-bit version that matches your system needs

![](media/image2.PNG){width="5.822916666666667in" height="1.3125in"}

2). Unzipping the binaries and extracting. At this step you could
inflate the binaries to the path location, but for this demonstration I
skipped it in order to show Terraforms behavior.

![](media/image3.PNG){width="6.105018591426072in"
height="2.5836942257217848in"}

3). Verifying that even unzipping the file, the path still points to the
v0.13 binaries

![](media/image4.PNG){width="5.875820209973753in"
height="1.5002088801399824in"}

4). Moved the new binaries to the folder created to house the previous
version binaries, after relocating the older ones. Not doing so will
force a rename of the executable, and will prevent Terraform from
recognizing the file.

![](media/image5.PNG){width="6.230036089238845in"
height="1.0522298775153105in"}

5). Test that this worked and that the system can call the binaries from
any location (the root of C). Note also I intentionally moved out of my
Downloads directory to the root of C volume, and was still able to call
the binaries.

![](media/image6.PNG){width="5.813311461067366in"
height="1.2293383639545057in"}

[MAC OS/Linux](\l)

Note: if running in a Linux distro you may need to run sudo apt-get
install zip -y and sudo apt-get install unzip in order to download the
zip and unzip app to unzip the binaries at step 3.

1). The first step requires that we navigate to the [downloads
page](https://www.terraform.io/downloads.html) and copy the link address
for the appropriate download, in this scenario I'll be installing on
Ubuntu.

![](media/image7.PNG){width="5.552858705161855in"
height="2.50034886264217in"}

2). Open your bash or terminal and run wget along with the link address

![](media/image8.png){width="10.175811461067367in"
height="2.239303368328959in"}

3). Next, we run the unzip command with the download and verify it
worked, as noted by the green terraform file (executable)

![](media/image9.PNG){width="7.542719816272966in"
height="1.3960279965004374in"}

4). Now we want to move the binaries into the appropriate folder
(Binaries for this example), and verify they are there. At this time
ensure there are no other binaries that share the folder or you maybe
forced to rename them (Terraform won't be able to find them if named
differently).

![](media/image10.png){width="4.530683508311461in"
height="1.1665212160979876in"}

5\) If you've set Terraform up before, then you proabably set the
enviromental variable. Which at this point you could run a terraform
-version cmd to check it's functionality and be done. This, however, is
my first time setting up in Linux so I'll continue with the process

\- Enter your nano editor by typing nano \~/.profile

\- Once in the editor type export PATH="\$PATH/.Binaries" press ctrl +
"x", save the buffer with Y, then press enter to exit.

\- To finish the process you'll need to run source \~/.profile

-Disclaimer: This may mess up your path so you won't be able to call any
of your commands from you bash. If this happens simply type export
PATH=\"/usr/bin:\$PATH\"

-And finely verify functionality.

![](media/image11.PNG){width="5.520831146106737in"
height="1.4583333333333333in"}

Example configuration

[Dependency Lock File](\l)

The last item to configure in the install process is the Dependency Lock
File. This is a new feature released with v0.14, locks in your provider
when the terraform init command is ran for the first time, and prevents
future automatic upgrades to the provider plugins when the terraform
init command is ran. Terraform then stores the provider version in the
.terraform.lock.hcl (which it uses later to check the version against).
The picture below shows some of the file.

![](media/image12.PNG){width="9.417981189851268in"
height="1.6564807524059493in"}

The dependency lock file is Terraforms way to reproduce runs in a
reliable manner. So just where would this be useful? Several scenarios
exist, within teams and remotely, and in automated environments.
Automation in Terraform automatically repeats the steps found in the
workflow (init, run, and apply), reproduces these steps multiple times,
and each time the chance of a new version of the provider plugin being
released increases. Hence the reason behind the creation of the lock
file. Below I run though some examples I used from [HashiCorp learning
tutorial](https://learn.hashicorp.com/tutorials/terraform/provider-versioning?in=terraform/configuration-language&utm_source=WEBSITE&utm_medium=WEB_IO&utm_offer=ARTICLE_PAGE&utm_content=DOCS)
including their learning repository, feel free to check it out for more
exercises if you find you still need more information on the topic.

[Overriding the Behavior:]{.ul}

A few options exist when configuring the version with the
.terraform.lock.hcl file. The default setting (where no value is added),
specify the version constrain within the provider block(s) in your
version.tf file, or commit the .terraform.lock.hcl file In .gitignore.

[The Default Setting]{.ul}: as stated above and shown in the previous
picture, no value is added to the provider block, and therefore
Terraform uses the most recent version of the provider plugin
(hashicorp/aws 3.25.0).

[Adding a Version Constraint]{.ul}: Here the engineer would specify the
version to be used when terraform init command is ran for the first
time. Shown in the pictures below as an example, we have two provider
blocks, and both have different version numbers to explain the process
more thoroughly.

1)  Version specified in the versions.tf file

> ![](media/image13.PNG){width="3.406725721784777in"
> height="4.531882108486439in"}

2)  Then terraform init

![](media/image14.PNG){width="7.709409448818898in"
height="3.8442869641294837in"}

.[gitignore:]{.ul} The third option is used simply to tell terraform to
ignore the .terraform.lock.hcl file by committing it to your repository
in a .gitignore file. This as you guess, tells terraform to ignore it
when the terraform init/run commands are used, and thereby reproducing
behavior similar to v0.13. In-fact. if the .lock file is not committed
to your repo, you will recreate behavior similar to v0.13, but still
requires the version argument in the provider block to set the version.
The highlighted section in the 2^nd^ picture shows this.

Finally, if you decide to use the dependency lock file the provider can
be upgraded later by running terraform init -upgrade, this will bring
the current version to the most recent provider plugins. Note that the
-upgrade flag will only work as long as the latest version falls within
the boundary of the specified constraint in your write. This is
specified with the "is greater than" (\>=) operator. If you later find
you need to downgrade to a lower provider plugin first specify a lower
constraint with a "is less than" (\<=) operator alongside the -upgrade
flag.

That concludes the download and installation process. Thanks for taking
the time to read through and I hope it answers some of the question you
had about the upgrade process.
