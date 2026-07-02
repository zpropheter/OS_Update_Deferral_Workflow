# Deferral Auto Update Workflow
![Software Update Settings]([https://imgur.com](https://imgur.com/02wvOhS))
## Requirements
- Jamf Pro
- Blueprints
- Apple Silicon macOS Device

# Installation

## Script and EA Setup
1. In Jamf Pro> Settings> Scripts paste the install_ld_policy.sh script and save
2. Update lines 68 and 261 with the number of days you are setting to defer (this will only affect reporting, not actual deferrals)
3. Create a Jamf Pro Policy containing the script and scope it to once per computer
4. In Jamf Pro> Settings> Computer Management> Extension Attributes create entries for major_version_due_ea and minor_version_due_ea
5. Allow the devices to go through an inventory update to create values for the EAs

## Smart Group Setup
1.  Create a smart group for enforcing deferrals with the following criteria
    1. Minor Deferral Due is ______
    2. or
    3. Minor Deferral due is None
2. Create a smart group for enforcing updates with the following criteria
    1. Minor Deferral Due is not ______
    2. and
    3. Minor Deferral due is not None

## Blueprints
1. Create a blueprint with Software Update Settings Payload
    1. Set the deferral in the blueprint to match the days set in Step 2 of the Script and EA Setup section.
    2. Scope the blueprint to the Smart group created in Step 1 of Smart Group Setup.
2. Create a blueprint with Software Updates Payload
    1. Configure the blueprint to use the following settings
        1. Enforcement Type: Latest OS Version
        2. Check the box to "Ignore Major Versions"
        3. In Days after release to enforce update include the number of days in the deferral and add that to how many days you want to allow the user to install the update. For example, if you defer updates for 7 days and want users to have 3 days to install the update, this value should be 10 (max value is 30 as of 7/1/2026)
    2. Scope the blueprint to the Smart group created in Step 2 of Smart Group Setup.

# Workflow
- By default, devices will fall into the deferred updates section to prevent unintentional updates. As the devices report in, they will update the values and either remain in the deferral group if they're up to date, or fall into the auto update blueprint if they have an OS Update beyond the deferred range
- The `Latest OS Update` enforcement ignores deferrals, however, at it's current state it will queue multiple updates in sequence rather than just jump to the latest version. This is normally an inconvenience, but for the sake of enforceing deferrals it allows us to only install the deferred version that has recently expired as noted below.
    - Example: A macBook running macOS 26.5 has a deferral for 7 days. 
    - Apple releases 26.6 and the 7 day timer starts
    - Before the deferral ends, Apple releases 26.7
    - The macBook falls into the blueprint for `Latest OS Update` and sees that 26.6 and 26.7 are available. With your 10 day requirement in the blueprint, this gives 3 days after the deferral to install 26.6 and 13 days for 26.7, both of which are queued for install
    - 26.6 enforces first and the user installs. At the next checkin, the device checks the EA again and sees that 26.7 is still within the deferral window and falls out of the blueprint group for `Latest OS Update` and back into the 7 day deferral blueprint which is now enforced. 

# Caveats
- The LaunchDaemon is set to run once a day and after a reboot to keep the values updated after an OS Update Event. 
- The EA will update at each inventory update in Jamf Pro. Because of this, it may result in the blueprint swap over taking up to a day after update depending on how aggressive your inventory update is for your server as a whole. 
