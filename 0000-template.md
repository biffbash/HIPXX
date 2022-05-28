# HIP Template

- Author(s): biffbash
- Start Date: 2022-05-07
- Category: Technical
- Original HIP PR: <!-- leave this empty; maintainer will fill in ID of this pull request -->
- Tracking Issue: <!-- leave this empty; maintainer will create a discussion issue -->

# Summary
[summary]: This HIP will provide Fixed time interval PoC (3 times a day) and remove randomness to provide a more transparent and consistent PoC
additonaly to also improve the currently very limited custom antenna options when asserting a device, it will increase the PoC limit distance to 150km and provide support for legal pre-amps and repeaters, enabling many users to witness Lora signals more effectively.

# Motivation
[motivation]: Randomness should not be a factor in rewarding the coverage you provide since the coverage your providing is not random and should be consistent as well,
this will enable transparency if your hotspot beacon fails at the designated time there should be a network failure report as to why and what stage it failed.

Helium network is sent via RF and your antenna setup should be as simple as a omni or as advanced as your are allowed in your region.
The current custom antenna selection leaves alot to be desired,  at the moment we only have one value for receive and gain together and a value for height.
This HIP aims to tackle such issues as :
 
 1) At the moment we only have one value for receive and gain together , Receive and Gain need to be seperate values.
 
 2) there needs to be a entry point for 
 A) Legal pre-amps(receive is amplified only) and
 B) Bi-directional amplifiers with **Automatic Gain control** set to or below their regions limit (TX is amplified ONLY if below a set DB threshold) 
_NOTE HERE: NEITHER A) or B) can exceed EIRP legal TX limits in either case._ 
 
 3) Support for using more than one antenna (ie using a different antenna for receive and a different one for sending) , these antenna may not always be identical gain  or receive , thus they require different entry points for DB gain and DB received. 

4) Support for Repeaters - Extending the network coverage is one of the primary goals of building the helium network , we should be embracing anything that can legitimately extend our coverage, as a side benefit it will increase the chances hotspots of witnessing a beacon that was your hotspot missed due to channel hopping.
The support for repeaters will include Extending the range in PoC limit to 150km as typical sensors with external aerials are capable of going over 100km. Also only for Repeated beacons a further increase to 200km from the orginal beaconer PoC only with a "repeated" flag added into the options field in the data packet to allow for the extra distance the repeater is capable of.

5) Fixed beacon interval and removal of randomness. No longer will you have an arbitrary waiting period, hotspots will beacon at its allocated time , 3 times a day 8 hours apart with little to no deviation.


# Stakeholders
[stakeholders]: #stakeholders

* Hotspot owners will be able to take advantage of these improvements if they choose , but many will see an increase in their earnings if others within range implement these now supported improvements into their antenna designs by potenitally increased witnesses from further distances.
Additionally for those within range of a repeater the likelyhood that your radio will miss beacons will be decreased and the rate of witnessing will increase.



# Detailed Explanation
[detailed-explanation]: #detailed-explanation

Removal of randomness in PoC, this will set beacons to 3 times a day(equivalent to PoC interval 480) at fixed 8 hour intervals this improvement will allow :-

A) a user to be able to quickly see if the network or the hotspot is responsible for the failure of a beacon at their 8hr exact time, network performance will increase if users are able to quickly notice if their unit has not beaconed in the last 8 hours.

B)The removal of the unnessecary randomness from PoC will provide a more fair and accurate and RELIABLE PoC process for all users.

The hotspot when a connection is established to a validator should start the 8hr PoC timer with a verfication process to the validator, and on the validator a check on the blockchain to see if the unit  has beaconed within the last 8 hrs (to dissaude gaming). 


The use of seperate antennas for receive and gain is not new, for anyone experienced in the RF field
 it is quite simple to split the receive and transmit signial from one SMA or N type connector.
here is a video simply illustrating one of many ways how that is achieved, this one is using circulators.
https://www.youtube.com/watch?v=lvA5VamV6Cs

To support the use of seperate antennas , under custom antenna there will be two fields to input for antenna gain DB and receive DB, there will be a optional field for pre-amp gain and an optional box for Bi-directional amplifier receive and lastly a AGC field for DB threshold cutoff and gain DB.

why these fields are required
custom  antenna gain and receive : using seperate antennas (eg one 8db aerial for receive and a 3db aerial for TX) putting your one current DB value as 3 will result in legitimate beacons being invalid as the RSSI will be too high, conversely  putting 8db as your one value  may result in your radios power being limited on transmit on a 3db aerial.

**Pre-amps & bi-directional amplifier support:**
there are many widely known products out there such as flarm booster and a plethora of others on ebay and alibaba
https://www.youtube.com/watch?v=JVWTKEUv5j0
Support for these devices that improve our network coverage and reliability should be adopted as they are currently widely used.


**Repeater support:**
Putting in a repeater  when a terrain obstacle is in the way of RF is quite often a great solution, it means that you will be able to legitimately transmit & receive data to and from sensors on the on the other side of the obstacle.
It also will extend the legitimate distance you can communicate with other hotspots and sensors.

Currently Repeaters can be used but only with a transmit delay on the same frequency, if it is rebroadcast on another frequency currently a  "witness on incorrect channel" error will be present as your radio has received the signal but not on the channel it was originally beaconed from.

This HIP proposes full support by changing a byte flag in the existing options field of the data packet when being rebroadcast by a repeater, to designate a repeated signal. this flag will make other repeaters ignore a packet that has already been repeated and stop a feedback loop (ensuring a signal is only repeated once).
additionally in the case of repeating on a different channel than the original signal channel the abovementioned flag would be checked against the incorrect channel witness as rebroadcasting on a different channel allowed by your region would now be suitable.

**PoC Distance increase** 

"Typical" lora sensors that have external antennas are more than capable of 100km. A more realistic figure is 150km for those with good line of sight elevation or are transmitting over water"

The founders of the lora standard, have successfully made a:
"LoRaWAN® distance world record broken, twice. 766 km (476 miles) using 25mW transmission power"
https://lora-alliance.org/lorawan-news/lorawanr-distance-world-record-broken-twice-766-km-476-miles-using-25mw-transmission/

HIP58 only brought cherry picked statistical data to backup their limit of 100km unfortunately. Neither did we see any correspondence or info from any actual manufacturers of helium sensors whom this HIP affects directly.

# manufacturer statements
[manufacturer-statements]: #manufacturer-statements
**Manufacturer statements.** to be added.
This HIP has reached out and contacted manufacturers such as "_company names yet to be disclosed_" and these are some statements from them:

# technical reasoning
[technical-reasoning]: #technical-reasoning

Technical reasoning for raising the PoC limit:

quick preface:
To put the power gains into scale for those who dont understand Decibels dB
"To produce an increase of +3 dB you simply need to double power (watts)" , eg 23dB signal is twice as strong as 20dB signal.  same as a 15db is half the power of an 18db signal.

Take an example that most radios will receive around the -132 db RSSI approximately until the signal becomes too weak 
Free-space path loss (FSPL) is the attenuation of radio energy between the feedpoints of two antennas that results from the combination of the receiving antenna's capture area plus the obstacle-free, line-of-sight path through free space (its how you estimate how strong a signal should be)

On a 100km TEST scenario (labratory conditions and as a disclaimer real world results will have more loss!)
europe : for 100km distance on 868Mhz  with 16DB transmit and 6DB receive the loss is 109.2 dB
US + AUS: for 100km distance on 915Mhz  with 30DB transmit and 6DB receive the loss is 95.67 dB 
CHINA: for 100km distance on 470Mhz  with 12DB transmit and 6DB receive the loss is 107.9 dB

when our approximate minimum signal to be received is at -132db ,if a 100km limit is imposed
you are theorectically limiting europe by 22.8 DB!
you are theorectically limiting USA+AU by 36.33 DB!
you are theorectically limiting china by 24.1 DB!

Thats quite alot of loss! , 
yes it is and to address this limit, this HIP will raise the limit to 150km for non repeated beacons to ensure fair and equitable rewards for those the previous limit impacted. 200km for repeated beacons as stated with the conditions above.
Furthermore current 100km limitation never affected gamers who very simply re-asserted themselves within 100km the current limit only slows the expansion of the network and doesnt reward device interaction at that range.

As a last note use of repeaters only strengthens the case for an increase to more realistic coverage limit and with the help of this HIP will more accurately reflect the real life antenna setup users have as users will have the ability to enter these details if required.

# Drawbacks
[drawbacks]: #drawbacks

- Why should we *not* do this?

# Rationale and Alternatives
[alternatives]: #rationale-and-alternatives

This HIP beleives in freedom of choice
We feel Users should be free to have any antenna setup (as long as its legal in respective region), the current custom antenna interface provides no options users to input their required values to match their actual antenna system.

That would be result in any user being easily able to provide superior coverage by improving their helium antenna system by using a AGC(auto gain control) amplifier or pre-gain amplifier(receive only amp) .
This is your chance to discuss your proposal in the context of the whole design
space. This is probably the most important section!

The biggest caveat in this HIP for most end users is that if anyone puts a repeater in your area your witnessing consistency is likely to increase, 

- Why is this design the best in the space of possible designs?

- What other designs have been considered and what is the rationale for not
  choosing them?

- What is the impact of not doing this?

# Unresolved Questions
[unresolved]: #unresolved-questions

We expect the interface  selection for the New custom antenna options to be simple.

- What parts of the design do you expect to resolve through the implementation
  of this feature?

- What related issues do you consider out of scope for this HIP that could be
  addressed in the future independently of the solution that comes out of this
  HIP?
  the distance limit should be governed by HIP-49 regional limits not HIP-58 with only ONE arbitrary number limit for every region regardless of power limits and  frequency loss are different for each region and we are supportive of incorporating function that into HIP-49 governance. 

# Deployment Impact
[deployment-impact]: #deployment-impact
 With the obfuscating randomness removed the network performance will be alot more transparent to all users .

Current users can re-assert their antenna settings under custom antenna.
a firmware update with support for the new custom antenna fields may be required so devices can display their information correctly, and the handling of the data packets flagged with a repeater byte will have to be supported.

- Is this backwards compatible?

        - If not, what is the procedure to migrate?
        - Roll back to pre LHS.

# Success Metrics
[success-metrics]: #success-metrics

legitimate users with over 100km range wont be adversely affected anymore, a WIN for the peoples network!

- What should we measure to prove a performance increase?

- What should we measure to prove an improvement in stability?

- What should we measure to prove a reduction in complexity?

- What should we measure to prove an acceptance of this by it's users?
