overall concepts:
* there are files that are currently being generated in our process when the requirement is in-progress status
  * These files get deleted, currently, once this part of the process is done. IT shouldn't.
  * These files should stay in "in-progress", but better organized - under a subdirectory - like how files in completed/ are stored
  * analysis-notes.md, sections-c-e.md, and implementation-summary.md should remain in completed/ under the requirement subdirectory
  * integration-spec.md should not be defined in it's own file, but included within sections-c-e.md
      * update requirement-template.md, readme, claude, etc. in order to reflect this.
* We need to audit and ensure these actually align.
  * Aime/workspace/requirements/GlobalRequirements/IndividualRequirements
    * DCS should have it's own section in here.
    * We want this to be the sole repository for specific details that can be referenced in the files below.
    * each md file / requirement should be specfic to one topic only to ensure granularity and concisiveness in this directory
      * If there are requirements that have more than one topic, break out the other concepts into their own requirement file.
  * /app/workspace/requirements/CLAUDE.md
  * /app/workspace/requirements/ProducerPortal/CLAUDE.md
  * /app/workspace/requirements/ProducerPortal/architectural-decisions.md
  * /app/workspace/requirements/ProducerPortal/entity-catalog.md
  * Aime/workspace/requirements/ProducerPortal/queue/README.md
    * These files should work as context builders and translators for files IndividualRequirements
      * Keeping these concise and in sync with IndividualRequirements allows for us to easily propigate concepts as well as update exisiint concepts
    * Take any detailed information out of these files that should be outlined as a global requirement
      * create a new global requiremnt or consolidate with an existing one so the information is not lost
      * ensure the information is still "available" via context/reference from where it came from
        * This should lead to a cleaner context file.
  * Aime/workspace/requirements/ProducerPortal/templates/requirement-template.md
    * this is mostly correct and I like the structire.
    * ensure this is just updated to not reference anything oudated.
    * ensure this is included in the readme or claude files as the template to output the requirements, including section c and e, in the final output.

Aime/workspace/requirements/ProducerPortal/queue/completed/IP269-New-Quote-Step-1-Primary-Insured/implementation-summary.md
Global Requirements Alignment Achieved
* Technically, Applied Global Requirements should also be Cross-Reference Validation if everything stays in sync
Implementation Architecture
* This type of information should fall within section C of the requirements.
* Ensure all supporting files like requirement-template.md, readme, claude ensure this gets captured in the requiremnts.

Aime/workspace/requirements/ProducerPortal/queue/completed/IP269-New-Quote-Step-1-Primary-Insured/integration-spec.md
* this should be broken out into a dcs global requirement and referenced in supporting files.

Aime/workspace/requirements/ProducerPortal/queue/completed/IP269-New-Quote-Step-1-Primary-Insured/analysis-notes.md
* keep this as a part of the process

Aime/workspace/requirements/ProducerPortal/queue/completed/IP269-New-Quote-Step-1-Primary-Insured/sections-c-e.md
* this should be more reflective of requirement-template.md

for the proposed Implementation Order
* We are needing to do all this now.
* Our glamplan should include all of this.


Outline the full gameplan in a new md file for approval. do not change anything yet.