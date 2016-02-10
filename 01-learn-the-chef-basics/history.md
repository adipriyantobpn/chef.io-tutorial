Learn the Chef Basics
=====================

Contents
--------
1. [Set up a machine to manage][2]
2. [Configure a resource][3]
3. [Configure a package and service][4]
4. [Make your recipe more manageable][5]


##Set up a machine to manage

1. Launch Ubuntu 14.04 virtual machine
    - [Set up your own server][6]
2. Prepare a text editor
    - Emacs : [Absolute Beginner's Guide to Emacs][emacs]
    - Nano : [The Beginner’s Guide to Nano, the Linux Command-Line Text Editor][nano]
    - Vim : [Interactive Vim tutorial][vim]
3. Ensure the apt-cache is up to date
    ```
    $ sudo apt-get update
    ```


##Configure a resource

1. Run & connect to the server
    ```
    $ vagrant up
    $ vagrant ssh
    ```
2. Install the Chef DK
    ```
    $ sudo apt-get install curl --yes > /dev/null
    $ curl https://omnitruck.chef.io/install.sh | sudo bash -s -- -c stable -P chefdk
    $ # --  command above is just as same as these commands below
    $ # curl https://omnitruck.chef.io/install.sh
    $ # sudo chmod +x install.sh
    $ # sudo ./install.sh -c stable -P chefdk -v 0.10
    ```
    download : [install.sh][install-chefdk]
    review : [install.log][install-chefdk-log]
    literature : [bash options][bash-options]

3. Set up your working directory
    ```
    $ mkdir ~/chef-repo
    $ cd ~/chef-repo
    ```
4. Create the MOTD file using Chef locally
    - Create chef `recipe` file
        ```
        $ nano hello.rb
        ```
    - Add these lines, then save the file using shortcut combination `^O -> Enter -> ^X`
        ```
        file '/tmp/motd' do
            content 'hello world'
        end
        ```
    - Execute the recipe using local mode `chef-client`
        ```
        $ chef-client --local-mode hello.rb
        ```
    - Verify if the MOTD file had been successfully created
        ```
        $ more /tmp/motd
        ```
    - Run again the command for a second time
        ```
        $ chef-client --local-mode hello.rb
        ```
        review : [execute.log][motd-create]
5. Modify the MOTD file's contents using Chef
    - Open the chef `recipe` file
        ```
        $ nano hello.rb
        ```
    - Modify the file, then save the file using shortcut combination `^O -> Enter -> ^X`
        ```
        file '/tmp/motd' do
            content 'hello chef'
        end
        ```
    - Execute the recipe
        ```
        $ chef-client --local-mode hello.rb
        ```
    - Verify if the MOTD file had been successfully created
        ```
        $ more /tmp/motd
        ```
        review : [execute.log][motd-modify]
6. Ensure the MOTD file's contents are not changed by anyone else
    - Simulate that MOTD file had beed changed by some person
        ```
        $ echo 'hello robots' > /tmp/motd
        ```
    - Run the chef recipe
        ```
        $ chef-client --local-mode hello.rb
        ```
    - Verify if the MOTD file had been successfully created
        ```
        $ more /tmp/motd
        ```
        review : [execute.log][motd-accident-modif]
7. Delete the MOTD file using Chef
    - Create `goodbye.rb`, then save it
        ```
        file '/tmp/motd' do
            action :delete
        end
        ```
    - Run the chef recipe
        ```
        $ chef-client --local-mode goodbye.rb
        ```
    - Verify if the MOTD file had been deleted
        ```
        $ more /tmp/motd
        ```
        review : [execute.log][motd-delete]

##Configure a package and service

1. Install the Apache package
    - Create chef recipe file
        ```
        $ nano webserver.rb
        ```
    - Insert these lines
        ```
        package 'apache2'
        ```
    - Execute the recipe
        ```
        $ sudo chef-client --local-mode webserver.rb
        ```
2. Start and enable the Apache service
    - Modify the `webserver.rb` file to be like this
        ```
        package 'apache2'
        #
        service 'apache2' do
            supports :status => true
            action [:enable, :start]
        end
        ```
    - Execute the recipe
        ```
        $ sudo chef-client --local-mode webserver.rb
        ```
3. Add a home page
    - Modify the `webserver.rb` file to be like this
        ```
        package 'apache2'
        #
        service 'apache2' do
            supports :status => true
            action [:enable, :start]
        end
        #
        file '/var/www/html/index.html' do
          content '<html>
          <body>
            <h1>hello world</h1>
          </body>
        </html>'
        end
        ```
    - Execute the recipe
        ```
        $ sudo chef-client --local-mode webserver.rb
        ```
4. Confirm your web site is running
    ```
    $ curl localhost
    ```

##Make your recipe more manageable

1. Create a `cookbook`
    - Create cookbook directory
        ```
        $ mkdir ~/chef-repo/cookbooks
        $ cd ~/chef-repo/cookbooks
        ```
    - Generate cookbook named `learn_chef_apache2`
        ```
        $ chef generate cookbook learn_chef_apache2
        ```
    - Review cookbook directory structure
        ![Chef cookbook directory structure][img-chef-cookbook-dir-structure]
2. Create a template
    - Generate HTML template for `learn_chef_apache2` cookbook
        ```
        $ chef generate template learn_chef_apache2 index.html
        ```
    - Review generated template file location in cookbook directory
        ![Chef cookbook directory structure with HTML template][img-chef-cookbook-template-dir-structure]
    - Modify HTML template file
        ```html
        <html>
          <body>
            <h1>hello world</h1>
          </body>
        </html>
        ```
3. Update the recipe to reference the HTML template
    - Modify default recipe file
        ```
        $ nano learn_chef_apache2/recipes/default.rb
        ```
    - Add these lines to the recipe
        ```
        package 'apache2'
        #
        service 'apache2' do
          supports :status => true
          action [:enable, :start]
        end
        #
        template '/var/www/html/index.html' do
          source 'index.html.erb'
        end
        ```
4. Run the cookbook & verify web server
    ```
    $ sudo chef-client --local-mode --runlist 'recipe[learn_chef_apache2]'
    $ curl localhost
    ```
    review : [execute.log][cookbook-log]

##Appendix: Set up your own server

1. Virtual development environment
    1. Install required applications (Virtualbox & Vagrant)
        1. Register to $PATH (for Windows only)
            - Virtualbox : `C:\Program Files\Oracle\VirtualBox`
            - Vagrant : `C:\HashiCorp\Vagrant\bin`
        2. Verify :
            - Virtualbox
                ```
                $ VboxManage --version
                ```
            - Vagrant
                ```
                $ vagrant --version
                ```
    2. Prepare environment
        1. Download an Ubuntu 14.04 Vagrant box
            ```
            $ vagrant box add adipriyantobpn/ubuntu-14.04_chef-provisionerless
            ```
        2. Bring up the instance
            ```
            $ vagrant init adipriyantobpn/ubuntu-14.04_chef-provisionerless
            $
            $ nano Vagrantfile
            $ # -- add this line before `end` line
            $ # config.vm.network "private_network", ip: "192.168.10.10"
            $
            $ vagrant up --provider=virtualbox
            ```
        3. Verify the instance
            ```
            $ vagrant ssh
            vagrant@vagrant:~$ exit
            ```
        4. Cleaning up
            ```
            $ vagrant destroy --force
            ```
            
[Back to Point 1][2]

[1]: #contents
[2]: #set-up-a-machine-to-manage
[3]: #configure-a-package-and-service
[4]: #configure-a-package-and-service
[5]: #make-your-recipe-more-manageable
[6]: #appendix-set-up-your-own-server

[emacs]:    http://www.jesshamrick.com/2012/09/10/absolute-beginners-guide-to-emacs/
[nano]:     http://www.howtogeek.com/howto/42980/the-beginners-guide-to-nano-the-linux-command-line-text-editor/
[vim]:      http://www.openvim.com/

[install-chefdk]:       resource-log/01-configure-a-resource/01-install-chefdk/install.sh
[install-chefdk-log]:   resource-log/01-configure-a-resource/01-install-chefdk/install.log
[bash-options]:         resource-log/01-configure-a-resource/01-install-chefdk/advanced-bash-scripting-guide_options.html

[motd-create]:          resource-log/01-configure-a-resource/02-create-motd-file/execution.log
[motd-modify]:          resource-log/01-configure-a-resource/03-modify-motd-file/execution.log
[motd-accident-modif]:  resource-log/01-configure-a-resource/04-simulate-accidental-file-modification/execution.log
[motd-delete]:          resource-log/01-configure-a-resource/05-delete-motd-file/execution.log

[img-chef-cookbook-dir-structure]:          resource-log/03-make-your-recipe-more-manageable/chef-cookbook-directory-structure.png
[img-chef-cookbook-template-dir-structure]: resource-log/03-make-your-recipe-more-manageable/chef-cookbook-template-dir-structure.png

[cookbook-log]: resource-log/03-make-your-recipe-more-manageable/execution.log