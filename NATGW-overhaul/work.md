# NAT Gateway Overhaul

### Introduction
During my time on Shopify's Restricted Infrastructure and Stacks team as a Production Engineer, I was tasked with overhauling the NAT gateways for our financial reporting systems. These NAT gateways enable internal resources to establish connections with external resources, while adding an additional layer of security by effectively concealing the original internal IP address of the source. This proactive measure prevents potential threat actors from infiltrating the internal network by channeling all traffic through a centralized point; a virtual machine designated as the NAT gateway. Both incoming and outgoing traffic are managed through the implementation of firewall rules, which govern the traffic permitted through the NAT gateway. In addition to the added security benefits, this configuration facilitates comprehensively logging of traffic entering and exiting our internal network, thereby greatly enhancing our network monitoring capabilities.

### Project Scope
The key objectives of the NAT gateway overhaul were as follows:
* Decouple the NAT gateway for each environment from Google Cloud Platform's (GCP) Instance Group Manager (IGM)
* Add labels to aid cost control monitoring of Google Cloud Platform resources
* Add new labels relevant to team-specific network resources
* Migrate all virtual machines (VM) to N2 machine types

### Challenges and Solutions
All the tasks described earlier needed to be executed across three VMs that were configured to function as NAT gateways, playing a crucial role in our infrastructure. Due to their critical nature, it was imperative to minimize downtime while completing these tasks. Moreover, the challenge was compounded by the requirement to carry out the work within a live production environment.

Our infrastructure featured three identical NAT gateways, strategically designed for enhanced availability. This design allowed for a systematic approach: one NAT gateway could be taken offline at a time. With the other two VMs seamlessly handling the traffic that the offline VM would typically manage, concerns regarding disruption were effectively mitigated. This strategy was complemented by regular updates provided to teams that might be affected, enabling them to delay deployments until the specific work in their regions was concluded.

### Technologies and Tools
Technologies and Tools used to complete this work are as follows:
* __Google Cloud Platform__
  - The cloud platform of choice for Shopify, housing all of our infrastructure.
* __Chef__
  - Used to configure and manage VMs within our infrastructure.
* __Terraform__
  - Used to manage the configuration of our infrastructure.
  - Terraform was used in conjunction with Github to serve as documentation of work completed, as required by SOX and PCI-DSS.
* __Github__
  - As mentioned previously Github was used in conjunction with Terraform to document our work.
  - Github was also used for it's typical purpose: version control.
  - When used with an internally-built continuous deployment tool, Github expedited the rollback of changes should issues arise.
* __SSH__
  - Used to securely gain remote access to another VM on the network, to monitor traffic to the NAT gateways.
* __Linux commands such as `watch` and `curl`__
  - Used to monitor traffic to and from the NAT gateways, from another VM on the network.

### Implementation
Here is a description of my approach to completing the work described above:

__1.__ Edit the existing Terraform configuration to make the desired changes. Create a new Github pull request (PR) containing these changes. By pushing the proposed configuration changes to a new PR, I was able to have a coworker verify the changes are correct before applying. Having the work approved by another member of my team also serves to meet the requirements of the PCI-DSS standard (change controls).

__2.__ Plan the changes locally to ensure they are as expected. Post plans within the body of the previously mentioned PR, for review and approval by senior team member.

__3.__ Work on one NAT gateway VM at a time, noting the IP address of the NAT gateway. As previously mentioned, working on one NAT gateway at a time allowed me to minimize disruption ensuring high-availability of our infrastructure.

__4.__ SSH into another VM on the network and test the connection to the NAT gateway using Linux commands. Because traffic to the NAT gateways was load-balanced, I would be able to confirm all three NAT gateways were receiving traffic.

__5.__ Delete the NAT gateway VM via Google console.

__6.__ Apply the Terraform configuration changes, using targets to single out one NAT gateway VM at a time.

__7.__ Check Cloud-init log files to ensure Chef has finished successfully. Confirming Chef had finished successfully means the new NAT gateway was ready to receive traffic.

__8.__ SSH to the previously accessed VM to monitor traffic to the NAT gateways. If the IP address of the re-created NAT gateway shows up, it means the NAT gateway was re-created successfully.

__9.__ Repeat this process for each of the remaining NAT gateways.

### Results and Impact
Here's a breakdown of the final results:
- The work was successfully completed with minimal dispruption and no downtime. 
- Our NAT gateways were removed from the IGM. This simplified our NAT gateway infrastructure, removing unecessary configuration which resulted in less time spent writing Terraform code, going forward.
- All of the labels were added fulfilling a request of our Platform Efficiency team. These labels enabled the Platform Efficiency team to track costs associated with our restricted GCP infrastructure.
- New network tags were added allowing us to update existing firewall rules.
- The VMs my team was responsible for, were all migrated to the N2 machine type. The migration to the N2 machine type provided better value and performance than the N1 machines we were using, previously.

### Lessons Learned
I learned a lot while working on this task. Here are some of my takeaways:
- Gradual progress with lots of communication is critical when dealing with a live production environment. The more communication, the better. Informing the developers of the work at each stage allowed them to avoid issues associated with deploying code changes to infrastructure while it was down. If I did not inform them of the work underway they may have wasted time troubleshooting issues which in reality, had simple solutions.
- I was unfamiliar with the method used to check the traffic to the NAT gateways, by connecting to another VM, before completing this work. It was incredibly valuable to learn this method, and it will be one I will use during similar tasks in the future.
- Including comments about decisions made and future work to be done within the Terraform configuration is important to long-term planning. Having as much context as possible helps inform future planning, tremendously.

### Future Considerations
One thing that stood out during the course of this work was the lack of documentation regarding decisions made, when the infrastructure was originally configured. There was little to no information about why an IGM was initially used for the NAT gateways. Even senior team members could not recall the reason this decision was originally made. While the IGM was ultimately deemed uneccessary for our existing infrastructure, having the historical context as to why it was configured this way would have helped us make a more informed decision about these changes. Consider some unforseen use-case in which some other aspect of the infrastructure was dependant on the NAT gateways being coupled to the IGM. The trickle-down effect of removing the NAT gateways could have caused outages or worse, failing a SOX or PCI-DSS audit.

### Conclusion
Navigating the intricacies of reconfiguring critical infrastructure within a live production environment was an enlightening experience. It's clear that robust planning and transparent communication act as safeguards against potential major outages. This experience significantly bolstered my confidence, solidifying the fundamental role of proactive strategies when configuring infrastructure.