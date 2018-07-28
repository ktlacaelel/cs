Abstract


    cs - check site

    The check site program checks if a site is online.


Problem


    Get notified when a site is down even if you are asleep.


Background


    There are many tools for this.
    Some of them come with different setup tax.
    Even with a lot of them in place sometimes is possible to miss a notification.
    We need a simple solution that is easy to plug to other kinds of notifications.
    Like sms, ittt, say, cloudwatch, home speaker system/lights, your apple watch buzzer.
    And also that allows for the quick addition of endpoints that need to be checked.


Constraints


	- reduce dependancies
	- ensure portability
    - avoid daemons
	- ensure dumb
	- super simple config
	- extendable
    - only accept 200 requests for now.


Dependancies


    - curl
    - ruby
    - bash


Hypothesis


	1. if we setup and abstract system which behavior is concise but interaction is configurable we should be able to achieve our needed goals.
	2. if we use microkernel architecture for this we should be able to establish the behavior but allow for use to extend the desired behavior.


Solution Proposal


    Have a directory with files that each have a url.
	Have a directory of hooks for failed cases and hooks for successful cases.
	Have a program that will read the urls test them and pass them to the necessary hooks.


Theory Of Operation


	A list of urls is read and tested for a 200 http status.
	If passed then it will be delegated to sucess hooks for handling.
	If failed then it will be delegated to fail hooks for handling.
	Sites are files with an url in them.
 	Hooks are shell programs that will receive the url as ARGV 1


Black Box


    sites
    fail hooks
    okay hooks
        cs
            check sites availability
            notifications N


Flow Diagram


    +------------------------+
    | user configures sites. |
    +------------------------+
         |
         V
    +------------------------+
    | user configures hooks. |
    +------------------------+
         |
         V
    +------------------------+
    | user executes program. |
    +------------------------+
         |
         V
    +----------------------+
    | program tests sites. |
    +----------------------+
         |
         V
    +-------------------------------------------------------+
    | program will execute hooks depending on the response. |
    +-------------------------------------------------------+


Functional specification


    hook (fail/okay):

    	Shell scriopt that will receive site url via ARGV 1

    site:

    	A file that will contain just an url.


Technical specification.


    ./bin/check_site program

    	use curl to check if the http status is 200
    	if not return 0
    	if yes return 1

    ./bin/check_sites program

        read list urls from the sites files
        for each site
            test site by calling check_site passing ARGV1 as site_url
            if site passes
                let hooks be a list of okay hooks
            else
                let hooks be a list of okay hooks
    	for each hook
            execute hook passing site as ARGV1


Notes:


    - Dont run daemons, they are chaotic.
        Use cron instead.
    - Use ix-log to manage output and properly stash data.
    - We don't care to enable disable hooks.
	  If its not being used we don't want it on the system.
      If it is what is the problem?
      We don't commit garbage, so when we checkout we don't get it back.
      that simple. have a nice day.

Example:


    ix and cron integration example.

    * * * *  cd /Users/kazu/dev/git/cs && ./bin/check_sites | /Users/kazu/dev/git/ix/bin/ix-log >> /Users/kazu/dev/git/cs/log/cs.log

	# sucess scenario:
    kazu@utopia:~/dev/git/cs$ echo 'http://example.com' > sites/example
    kazu@utopia:~/dev/git/cs$ > log/cs.log 
    kazu@utopia:~/dev/git/cs$ tail -n100 -f log/cs.log | ix json-key timestamp trace_id token_id message
    1532810880.9324002 ee29c3ee710d194f68dc51f0a14b7de5 c06489221d9a6e4191566194a5a91bbf ix log initiation marker
    1532810880.932555 ee29c3ee710d194f68dc51f0a14b7de5 c862cc64071c2f9a4848b4e046b39360        sites/example ./bin/check_site http://example.com
    1532810881.020201 ee29c3ee710d194f68dc51f0a14b7de5 851975a2b7960140bb9d086219b68674        sites/example is site okay? true -> http://example.com
    1532810881.02033 ee29c3ee710d194f68dc51f0a14b7de5 e6d053ea3d02f24f93dd7d1b3fc259bf        sites/example run hook: hooks/okay/print.sh http://example.com
    1532810881.029645 ee29c3ee710d194f68dc51f0a14b7de5 5e04e19869902baede900ebf73575ced        sites/example | site is okay: http://example.com
    1532810881.03295 ee29c3ee710d194f68dc51f0a14b7de5 debba9a57fe3b6918765be714b990469 ix log termination marker

	# failure scenario:
    kazu@utopia:~/dev/git/cs$ echo 'http://example.fail' > sites/example
    kazu@utopia:~/dev/git/cs$ > log/cs.log 
    kazu@utopia:~/dev/git/cs$ tail -n100 -f log/cs.log | ix json-key timestamp trace_id token_id message
    1532811060.875212 8a269decbb8674618d3cbdd1ec66c836 58cec8c136de4e7a8bc335f82bab1a91 ix log initiation marker
    1532811060.875324 8a269decbb8674618d3cbdd1ec66c836 a6410e4d3f6ca20a07d5eefb99db1480        sites/example ./bin/check_site http://example.fail
    1532811060.978886 8a269decbb8674618d3cbdd1ec66c836 fb45b4b4d05e22bf80b1390ff03f1409        sites/example is site okay? false -> http://example.fail
    1532811060.9790158 8a269decbb8674618d3cbdd1ec66c836 58dc7dad46c2cfb0a913cab2618aa2d9        sites/example run hook: hooks/fail/print.sh http://example.fail
    1532811060.987455 8a269decbb8674618d3cbdd1ec66c836 6a61ebb2d917eae3e347157021c0b99b        sites/example | WARNING: SITE DOWN http://example.fail
    1532811060.987552 8a269decbb8674618d3cbdd1ec66c836 408ce42826b24b7cf0cd50905739fbd4        sites/example run hook: hooks/fail/say.sh http://example.fail
    1532811067.305981 8a269decbb8674618d3cbdd1ec66c836 62968539789af3717477608a3ffec3b7 ix log termination marker


Future Work


	- run tests in parallel, the more sites the more the execution time
      will take and the more likely user will land to on overlaping worlds.
	- use a proper ix integration when it is released.


end
