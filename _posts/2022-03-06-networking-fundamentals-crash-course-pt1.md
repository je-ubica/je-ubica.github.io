---
layout: post
title: Networking Fundamentals Crash Course Part 1
excerpt: An Overview of the OSI Model Followed by an In-Depth Write-Up on the Physical and Data Link Layers.
category: networking
---
Heads up, this is long winded so the table of contents will preserve your sanity. This is also going to be geared towards the fundamental operations and not a guide on configuration. Configuration guides and man page level explanations are everywhere, including the respective vendor's websites, but it's a lot harder to find detailed explanations of how everything works and what to be aware of at a deeper level.

Essentially I'll be covering what you may see in passing on a vendor's official cert guide, perhaps quizzed briefly on their cert exams, but may forget in time. This also may be helpful for those outside of network engineering since these fundamentals are used every time two devices communicate on a network even though they're not at the forefront of sysadmin, programming, infosec, and all other work within the vast IT fields.

# Table of contents

- [Prefatory Ramblings](#prefatory-ramblings)
- [OSI Model](#osi-model)
  - [Layer 1 - Physical](#layer-1---physical)
  - [Layer 2 - Data Link](#layer-2---data-link)
  - [Layer 3 - Network](#layer-3---network)
  - [Layer 4 - Transport](#layer-4---transport)
  - [Layer 5 - Session](#layer-5---session)
  - [Layer 6 - Presentation Layer](#layer-6---presentation-layer)
  - [Layer 7 - Application Layer](#layer-7---application-layer)
- [Layer 1 - Physical Layer in Depth](#layer-1---physical-layer-in-depth)
  - [The Purpose of Layer 1](#the-purpose-of-layer-1)
  - [Digital Signals: Binary Logic, Bits, and How They're Carried](#digital-signals-binary-logic-bits-and-how-theyre-carried)
    - [A Detour into Computer and Electronics Development](#a-detour-into-computer-and-electronics-development)
    - [Back to Bits](#back-to-bits)
  - [Physical Layer Issues](#physical-layer-issues)
    - [Damaged or Broken Equipment](#damaged-or-broken-equipment)
      - [Electrical Damage](#electrical-damage)
      - [Physical Damage](#physical-damage)
    - [Environmental Issues](#environmental-issues)
    - [Hardware Limitations](#hardware-limitations)
      - [Cable Length and Attenuation](#cable-length-and-attenuation)
      - [Backplane Speeds](#backplane-speeds)
      - [Inherent Latency from Nodes and Distance](#inherent-latency-from-nodes-and-distance)
- [Layer 2 – Data-Link Layer In Depth](#layer-2--data-link-layer-in-depth)
  - [MAC Addresses](#mac-addresses)
  - [Switching](#switching)
    - [Layer 2 Segmentation with VLANs](#layer-2-segmentation-with-vlans)
    - [Issues From Lack of Layer 2 Segmentation: Loops](#issues-from-lack-of-layer-2-segmentation-loops)
    - [Native VLANs](#native-vlans)
  - [Layer 2 Security and Network Stability Concerns](#layer-2-security-and-network-stability-concerns)
  - [Layer 2 Traffic Types](#layer-2-traffic-types)
- [TCP/IP (DOD) Model's Link Layer: Yes I Lied About Skipping This](#tcpip-dod-models-link-layer-yes-i-lied-about-skipping-this)
  - [Why I'm Covering It Now](#why-im-covering-it-now)
    - [Francis Ford Coppola Presents  IEEE 802: Redux](#francis-ford-coppola-presents--ieee-802-redux)
  - [Media Access Control](#media-access-control)
    - [The Structure of Frames and How to Delimit Them](#the-structure-of-frames-and-how-to-delimit-them)
    - [Layer 2 Frame Portion](#layer-2-frame-portion)
    - [Layer 1 Frame Portion](#layer-1-frame-portion)
    - [Establishing an Ethernet Link: Speed, Duplex, and Auto-MDIX](#establishing-an-ethernet-link-speed-duplex-and-auto-mdix)
    - [Collisions and How To Stop Them with CSMA/CD and CSMA/CA](#collisions-and-how-to-stop-them-with-csmacd-and-csmaca)
  - [Logical Link Control](#logical-link-control)
    - [802.2 LLC and SNAP](#8022-llc-and-snap)
    - [Muxing on Shared Media](#muxing-on-shared-media)
    - [Flow Control and Error Handling](#flow-control-and-error-handling)
  - [TCP/IP Link-Layer & OSI Layers 1+2 in Summary](#tcpip-link-layer--osi-layers-12-in-summary)
    - [Common Physical/Link Issues and Things to Keep in Mind](#common-physicallink-issues-and-things-to-keep-in-mind)
- [Wrapping Things Up for Now](#wrapping-things-up-for-now)
  - [A Change of Plans](#a-change-of-plans)
  - [Moving Forward](#moving-forward)

# Prefatory Ramblings

This is something I've wanted to write for a while now and I think it will be a great first post to kick off this blog.

From my experience a majority of people in this field only know how things work in networking at a basic level, with their growing experience just adding breadth of knowledge and not depth of knowledge. For example, I'm admittedly pretty bad at math. I could do it accurately enough throughout school and college to get decent grades, but the pace I worked at was abysmal because I didn't truly understand the mechanics of every step taken to solve an equation. The slowdown was always due to trying to dig deep and stumble on the fleeting memory of "how" it works to use the equation correctly  because rote memorization never gave me the chance to learn "why" it works.

_I'm so happy the answer or guide to doing the rare math problem in day to day life is a quick internet search away._

Take the core of my shortcomings in math and change the subject to networking and I see examples of that every day from people I work directly or indirectly with, customers, vendors, and myself. That doesn't necessarily mean they're bad at what they do they just never had the resource or time or external push to really dive into the topic and hit every level of Bloom's Taxonomy for it.

This has been stuck in the back of my mind and gets reinforced every time I'm given "proof" of an issue that is only seen and felt with ICMP, every time traffic dies on a traceroute at hop _n_ when hop _n+1_ turned out to be missing a return route, and having to prove to a vendor with enough debug logs and packet captures fully explained with the relevant IETF RFC citations  as if I'm required to write a dissertation so the assigned TAC engineer doesn't close the case. Then I came across the following tweets that validated all of my thoughts and feelings.

<blockquote class="twitter-tweet" data-dnt="true" data-theme="dark"><p lang="en" dir="ltr">In networking if you understand the protocols you can figure literally anything out. If you memorized trivia about them you’re doomed.</p>&mdash; Stainless Steel DC Rat (@scarynetworkguy) <a href="https://twitter.com/scarynetworkguy/status/1345427678762033152?ref_src=twsrc%5Etfw">January 2, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

I seriously love this quick exchange, and seeing all the discussions like this one interspersed with genuinely good info and tutorials from people like Nixcraft, Swift on Security, Foone, and countless others gave me the drive to start sharing what I've picked up over time. Alright. Diatribe over. On with the show.

# OSI Model

Let's start off with some context for everything we'll go over. I'll ignore the TCP/IP(DOD) model as OSI is more specific.

## Layer 1 - Physical

This is all the physical wiring or RF. Data itself is transmitted in binary.

## Layer 2 - Data Link

Now we get into the first layer of encapsulation. Data is encapsulated into frames and moved by switches based upon source and destination MAC addresses

## Layer 3 - Network

The frames from layer 2 are encapsulated into packets and moved based on source and destination IP address routing.

## Layer 4 - Transport

More encapsulation, this time the layer 3 packets are encapsulated with TCP and UDP port numbers, as well as some oddball protocols outside of those two like SCTP and DCCP. Instead of packets we now have datagrams or segments, depending on the protocol.

## Layer 5 - Session

Here we start to get less network transit related and more end-user program related. The session layer deals with opening and closing sessions between different nodes, for example opening up an SMB connection between a PC and a Server to access a share, while SMB itself does not act here the transport via NetBIOS is what is Layer 5 specific, the RPC portion.

## Layer 6 - Presentation Layer

Here the data exists in whichever proper syntax the program should read it in. Raw ASCII input and output would be presentation layer while an HTTP page would not be until you view the page's source. Telnet is generally regarded to live at this level and a good example would be to create a telnet session to a device using Python's telnetlib where every input and output is given in a string such as "foo \n" as opposed to how you are seeing that same session in a telnet client that will do some extra legwork for user experience when you hit enter at the end of a string you typed out, it will transmit that string and include the newline character itself.

## Layer 7 - Application Layer

This is the actual data you would expect software to read and interpret. To circle back to HTTP, the full HTTP message with headers and data would be layer 7. Yes it can be encoded and transmitted in raw text but the difference here is the data is formatted in a way that it is sent and received in a meaningful way for the program.

Keep the OSI model in the back of your mind as the relationships between every layer will have you approaching things logically. I've seen people run in circles believing a device not sending SIP traffic _at all_ is due to the wiring even when the packet captures show good DNS queries to the public internet and the people trying to blame faulty wiring are remotely connected to the command line of the device without issue. Can't have layers 2-7 if layer 1 is down.

![OSI Model Encapsulation](https://www.computernetworkingnotes.org/images/cisco/ccna-study-guide/csg25-02-osi-encapsulation.png)

<cite>Above diagram sourced from https://www.computernetworkingnotes.com/ccna-study-guide/data-encapsulation-and-de-encapsulation-explained.html </cite>

# Layer 1 - Physical Layer in Depth

The physical layer is going to be something I'll try and write on more in depth in different posts because I feel that going deep into it has niche applications. Instead of covering the ins and outs of how single mode and multi mode fiber works, TDM vs DWDM, wiring specs, T-carrier circuits (although I have a strange need to talk in depth about them thanks to personal experience working with them), etc, I will  however cover the basics on what things are and go deeper into how copper ethernet works. My goal is to give good foundational knowledge so the familiarity becomes the tool to find more in depth documentation if you choose to. Cultivate the Google-fu.

## The Purpose of Layer 1

For all forms of communication there is a crucial problem (maybe design challenge?) that needs to be solved first. To use in-person verbal communication as an example its not going to be learning your first word as a child. Language is complex. There's structure and nuance to communicating in a spoken language. It's not morphemes either which are the basic sounds and tones that, when chained together, can make words because there's still logical structure to them.

The first step towards verbally communicating is being is the ability to create a sound that someone else can hear. I can't communicate verbally in English to the person next to me if I am mute, nor can I communicate verbally in English when that person next to me is deaf. Communication, before any logic or complexity is added, hinges on physically being able to move that information from one location to another, including from my mind to your mind. This is why the OSI model starts with the physical connections and transmissions as it's first layer.

## Digital Signals: Binary Logic, Bits, and How They're Carried

Now that we know the bare minimum requirement for communication to take place we can apply this to digital communication. I like to approach digital communication with two categories of connections, wired connections and wireless connections. Since digital wireless communication didn't kick off until the 90's we can approach digital wired communication first as it's safe to say wireless built upon the groundwork laid by wired.

Starting with the physical components to make communication happen we can draw parallels to the verbal speech analogy in the previous section:

- The Network Interface Controller (NIC) of the sender takes the role of the voice box
- The Network Interface Controller (NIC) of the receiver takes the role of the ears
- Electrical signals in the form of bits takes the roll of the sounds made the voice box
- Some form of conductive wiring to use as a medium takes the role of the air the verbal sounds travel through

For demonstration we'll use the standard IEEE 802.3 Ethernet and Category 5e twisted pair cables as our media type. It's the most commonly used and I'd imagine everyone reading this has some familiarity even without really knowing how it works, if you plugged a computer into your home router you've used both.

The defining part of Ethernet and why it outgrew and replaced older standards like token ring is that Ethernet combines sending and receiving across a shared media. This means a single NIC is able to handle input *and* output by itself using one cable. So what exactly is a digital signal and why is a shared medium so important?

### A Detour into Computer and Electronics Development

To understand a digital signal transmission I find it helpful to understand what the signal is first. The most basic form of information in all digital electronics is binary digits, or bits, which are the 1's and 0's. What those 1's and 0's mean is true or false, on or off, etc. Basically it's two possible states, think of a light switch being flipped on as the binary state "1", "True", or "On" while the light switch being off as the binary state "0", "False", or "Off". Digital electronics work using direct current electricity applied to circuits.

These circuits can have any number of components at the board level with wildly different layouts but they all accomplish their tasks by manipulating DC in a way to logically flip the right switches in the right order to get a desired effect.

Computers first started as mechanical devices like old school cash registers which could only do basic addition based on the keys pressed or naval fire control computers which were 3000 pound behemoths with enough cranks, levers, and gears inside it to make Rube Goldberg blush. Then came Electro-mechanical computers that could start doing more complex math by removing some of the needed cranking and lever pulling by using simple electrical relays and switches not unlike a complicated and oversized light switch.

Eventually vacuum tubes were able to replace the giant mechanical switches, to oversimplify these worked like light bulbs where the filament, the cathode, would start emitting electrons directed at a plate, the anode, when it got hot enough to operate but could start and stop signal output with a screen of wire between the filament and plate, the control grid, which acted as a gate. Applying a positive charge to the screen of wire allowed electrons from the cathode to pass through it and reach the anode like turning a switch on. Applying a negative charge would repel electrons like turning a switch off.

The reason I bring all of this up is the sheer amount of mechanical power or electrical power wasted just to flip the switch so to speak and the size of these components made the development of digital electronics unfeasible.

Then came MOSFETs. The metal-oxide-semiconductor field-effect transistor accomplished the switching and gate keeping done by earlier computers but at the fraction of the size. Instead of a vacuum tube that needs to draw enormous amounts of power in comparison just to get a controllable gate that can be switched on and off rapidly, MOSFETs work by having the two pins of it's switch, the source terminal and destination terminal, be physically bridged by layers of silicon and oxidized silicon.

There's a third terminal that connects to the gate and body of the MOSFET. When enough positive voltage is applied to this third terminal that it hits it's threshold the structure of the silicon changes to allow a channel through the conductive oxidized silicon and connects the source and destination terminals and allows electron flow. With MOSFETs the gate that controls the metaphorical light switch can now be as small as you can reliably manufacture it and flipped as fast as you can change the voltage.

This is what kicked off digital computer development and allowed the metaphorical switches be microscopic.

### Back to Bits

With MOSFETs in mind we can go back to what a bit actually is and how it's used. As stated before the detour in computers development no one asked for, the bit itself is a logical state of 1 or 0. Based on the pattern of 1's and 0's it can be abstracted by higher layers into useful data.

The patterns themselves are grouped together into 4-bit nibbles and 8-bit bytes, so two nibbles to the byte. Depending on the actual pattern of bits in each byte you get a certain numeric value. Right now we'll avoid the specifics in series of bits make what value, but once I finish the Layer 3 post and deploy it-- that can be your source of truth.

When it comes to the actual transmission of the information stored as the certain value in bytes, we work with the bits. Think of it like writing on a page, you can't really take your pencil and write the entire word at once, you need to write each letter in series. The bytes are the written words and the bits are the letters...although two bytes make a word in CPU architecture and I've already gone off the rails enough. _I'm sorry._

The transmission of bits over a CAT5e cable functions like taking a number of MOSFETs to be able to send a varying amount of voltage. Across the cable between each NIC there is mid point of voltage which acts as the reference point. This is just for connectivity not whether or not a 1 or a 0 is sent. When the first NIC wants to transmit a binary 1 to the second NIC it will increase the voltage above the reference point for one unit of time. To make sure that was an intentional increase in voltage, thus intentional data being transmitted, it needs to meet or exceed the voltage output high level.

On the flip side if the first NIC wants to transmit a binary 0 to the second NIC it needs to decrease the voltage to meet or exceed the voltage output low level. To my understanding reading 802.3 VOH is around 2V and 2.40V depending on the specific hardware and cabling used, and VOL is around 0.4V and 0.8V. The first NIC transmitting 10101010 would be using it's output transistors as switches to increase voltage to VOH for one unit of time, down to VOL for the next unit of time, back up to VOH, back down to VOL, so on and so forth like a rapidly flipped light switch.

The second NIC on the other side will take each peak and valley of voltage in sequential order and pass it into memory by using it's transistors which will activate on VOH to flip whatever switch to read the 1 bit into memory and it's other transistors which will activate on VOL to flip whatever switches to read the 0 bit into memory. Once enough bytes have been received the higher layer processes can do their job to convert the string of bits into useable information and act on it.

To be able to increase the speed of transmission as well as enable full duplex transmission, sending and receiving simultaneously, with copper we need to increase the number of wires and tighten up the voltage difference between VOH and VOL to flip the metaphorical light switch faster. While CAT5e has 8 pins for four total pairs of wires, if the NICs on each side are using 10Base-T they are actually just using one out of the four pairs and flips between 2.5V above the reference point and 2.5V below the reference point at the rate of 10 Megabits per second, or 10,000,000 flips of the metaphorical switch a second. The rate in which the bits are transmitted is baud, which is how many symbols (pulses or changes in that wave form) are sent per second.

It's only using one pair of wires because Ethernet tries to be as backwards compatible as possible while also future proofing, so 10Base-T was introduced with a single twisted pair being used as that was the most common wiring at the time but Category 3 cables released alongside it had the same four pairs as modern CAT5e and CAT6.

The increase in speeds to 100Base-T then narrowed the voltage differential to +1V/-1V but also introduced 4B5B encoding. The problem with going faster and tightening that voltage is the two NICs can get out of sync with each other, their clock rate gets skewed because they need to synchronize sending and receiving but based off their own connection to the other NIC. 4B5B encoding fixes that issue by mapping 4 bits of data that will be transmitted into a 5 bit code so that the metaphorical switch can be flipped faster but with the 4B5B encoding induces a clock signal by forcing transitions between VOH and VOL at least twice in every 5 bits sent to prevent each NIC from skewing out of sync if the data being sent is just binary 1's for a solid second.

Think of the clock skew like two people communicating with buzzers at a set tempo. Buzzing on and off 101010 is easy to keep track of buzz versus silence, but if there is a long period of 111111111 or 0000000000 at some point you can lose track of the tempo and interpret that as 7 buzzes in a row when it was actually 9 buzzes. Then when you start making those kinds of errors you can buzz off beat from the other person or at a slightly different speed. Encoding the buzzers to force it to go silent after two consecutive buzzes or make a buzz after two consecutive silences with a dictionary to reverse that encoding.

[It's a lot easier to visualize this with the 4B5B table so Wikipedia to the rescue.](https://en.wikipedia.org/wiki/4B5B#Encoding_table) The downside to 4B5B encoding is that 1 out of 5 bits are wasted, but to offset this the speed in which the metaphorical switch can be flipped is 125MHz, or starting at the mid point reference it can push the voltage up to VOH then down past the midpoint to VOL then back to the reference point 125,000,000 times a second. Factoring in the waste from the encoding it comes down to 100MHz or 100Mbps.

The next step from 100Base-T at 100Mbps is 1000Base-T at 1Gbps. This is when all four pairs of wires get used so its like having 4 10Base-T connections at once, but that's only 400Mbps not 1000Mbps, where does the other 600Mbps come from? Well we add in more points in those wave forms to trigger more metaphorical switches. Instead of having to swing the full +1V to -1V to get from triggering a 1 to be read by the receiving side to a 0 being read by the receiving side there's mid points between the +1 and -1 for a total of 5 different coded states to represent our binary states of 1 and 0.

In one cycle or unit of time we can transmit transmit one bit or symbol and have it come be seen as two consecutive bits being transmitted. With the additional wire pairs a matrix of voltage points are sent each cycle that comes out like full byte. So two bits per voltage point multiplied by 4 pairs gets 8 bits. There's more going on involving trellis modulation to add redundancy to this which if you really wanted to know is explained in [this pdf from UNH IOL](https://www.iol.unh.edu/sites/default/files/knowledgebase/ge/pcs.pdf), but this boils down to 125MHz per pair like 100Base-T multiplied by 4 for each pair and then multiplied by 2 from the additional voltage states for 1Gbps.

## Physical Layer Issues

Now that we have a decent understanding of what Layer 1 does, the parts involved, and what those parts actually do we can apply that to the most common physical issues.

### Damaged or Broken Equipment

This is the obvious one, either the device itself or the cabling between two devices has failed. Within this category there are two main causes:

- Electrical Damage
- Physical Damage

#### Electrical Damage

Electrical damage can be caused by a number of things like lightning strikes, brown outs, or faulty wiring. The damage itself comes from increasing or decreasing the voltage that's drawn into the equipment.

When a building takes a lightning strike and the electrical energy is able to to pass into the electrical system it spikes the voltage of everything connected to that system. The electrical system of the average US home is 120V Single Phase AC at around 100-200 amps depending on the size and needs which then gets broken off into 15 and 20 amp circuits at the breaker. That means whatever you plug into the wall is designed to take 120V AC, if that device has any kind of electronics in it that AC needs to be converted to DC with some form of power supply.

The issue when the lightning strike makes its way to the electrical system of the house it dumps tens of thousands more amps almost instantly into the wiring which flows into everything connected to it. The saving grace here is if the surge in power happens before a circuit breaker it will get tripped and protect the devices. If the surge is fed into the wiring after the home's circuit breaker a power strip that is an actual surge protector with it's own circuit breakers or fuses it can still provide that same protection.

On the other side there's brown outs. If the 120V AC fed from the utility pole drops down in voltage for whatever reason then the entire home's electrical system is operating at a lower voltage. In this case circuit breakers and surge protectors can't save your devices, only an uninterruptible power supply can and honestly the damage caused by a brown out without a UPS can be worse than a power surge.

The key thing in both of these scenarios is understanding Ohm's Law. Current = Voltage divided by Resistance, and thanks to basic algebra that means Voltage = Resistance multiplied by Current and Resistance = Voltage divided by Current.

- Voltage is the rate of electron flow measured in volts from the positive terminal to the negative terminal
- Current is the number of electrons measured in amps that are actually flowing from the positive terminal the negative terminal
- Resistance is the opposition to the flow of electrons measured in ohms between those two terminals

There's going to be an electrical engineer reading this that will say I'm wrong and in actuality voltage is the difference in electric potential and this is all about the flow of electric charge so electrons or protons can be flowing in either direction depending on the potential difference so there's no actual defined polarity and which ion flows in which direction depends on the actual application but als-

![quantum screaming](assets/images/screams-in-quantum-field-theory.png)

You are right and I hate you with the utmost respect because that goes over my head. I'll stick with the technician's understanding.

Back on topic. In the lightning strike scenario the amperage is increased. Since the wiring itself doesn't change to add more resistance to this increase in current the voltage would also increase since current * resistance = voltage and we increased current and kept resistance the same so voltage also had to change. That means a power supply that's connected at the time of the strike gets an increased current and voltage pushed into it, just like the wiring of the house the wiring of the power supply hasn't changed so resistance didn't either.

The problem happens in the internal components of the power supply like all the transistors, and capacitors can't increase their electrical output at the rate electrical input is being forcibly increased, or if the power supply can keep up and outputs high enough DC then the device's own internal components definitely won't be able to keep up with the increased rate. Ohm's law will then rear it's head and says SOMETHING needs to increase along with current because welcome to physics which must be obeyed, and it does so in the form of increasing resistance.... with heat.

The components can't increase the flow of electrons to keep up with the amount of electrons being shoved into that flow so the wires and capacitors and whatever else inside that device will heat up until they physically fail. Things start melting, resistors will burn and crumble, capacitors will straight up explode. That's the electrical damage.

The reason I say brown outs are worse is through that same logic. When voltage is decreased but fundamentally the components of the power supply or device itself don't magically change their physical resistance the only way to balance out the equation is increasing current drawn by the device or it's components. Voltage can dip down on that home's electrical service just enough to forcibly increase current throughout the house but as long as it doesn't increase the current on a particular breaker above 15 or 20 amps it won't ever flip and cut off that portion of the house for safety.

That means the device is still connected to an outlet that's dipping below 120V with increasing amperage and Ohm's law will again emerge from the shadows and say "hey, something needs to balance the math here" and thermal resistance happens like in the lightning strike scenario and components will pop and let the magic smoke out.

#### Physical Damage

While electrical damage does cause physical component damage, I'm more talking about things like dropping equipment. I'm not going to explain the science behind hitting a router with a hammer and why that's bad. What I will say is something that is so simple it can't possibly be the issue, right? Cable abuse.

Sure we all know about the Great North American Fiber Seeking Backhoe.

![fiber backhoe](https://i.imgur.com/D7POtlV.png)

The cable abuse that we don't think of first is far simpler. Pinching, cutting, or breaking a CAT5e cable enough that it doesn't fully break connection, but the connection is degraded beyond use. When talking about the twisted pairs of wires in the [Back to Bits](#back-to-bits) section there's something I didn't touch on and I really won't explain in detail until after the Layer 2 section. With connections like 1000Base-T two NICs can establish a connection with each other fine with just a two pairs of wires, but data isn't transmitted on just a two pairs.

More than once I have worked with customers that had  200Mbps or 1Gbps dedicated circuits fed to a colocation that patched to their equipment inside their cage and while the circuit and everything I had visibility and control over was fine, and same goes for the customers side, the connection was half down and it spewed enough errors that it may as well have been. The cause of the issue each time? The patch cable was pinched by the door of the cage, squeezing the twisted pairs inside the sheathing enough that almost all of the pairs were severed but not completely. The sheathing looked fine along with the two pairs required for both devices to "connect".

The entire time everyone was pointing fingers at potential causes, your device failed. No my device is fine *your device* failed. No mine is fine too. Okay then the last mile carrier has an issue. No the last mile carrier has perfect connectivity all the way to where they mux the 1-10Gbps fiber circuit at the colo to feed a number of customers and devices there, plus everyone else is working fine so it can't be the circuit. Eventually the customer who has the legal ability to open their cage sends someone down there and they see the cable pinched between the door and frame of the cage.

One new patch cable fed correctly into the cage later, everything is working fine. We eventually forget that something that simple can happen.

### Environmental Issues

I'll make this one brief. This is a barrage of information and we're still at the first layer.

There's three main things that can disrupt or harm devices based on the environment they are in:

- Heat
- Water
- Signal Noise

Water is the easy one, while pure water itself doesn't conduct electricity *everything that could be dissolved in water definitely will*. This is the bane of my existence in the service provider world. When a device gets water inside of it and that water creates a path between components to allow electricity to flow where we absolutely do not want it to flow things will short out and die. However, that is not the main cause of my pain. It's not super common for water to get inside of a room, and certainly not to the point it gets past the housing of the device and onto the circuit board. My torment comes from wet copper circuits.

The same principles apply here in regards to how the damage is done, but did you know the telecommunication cables used to be insulated with paper? I do. The remaining T1 circuits I still support, as of the time I'm writing this, definitely know they're insulated with paper. Every time it rains at least a dozen become saturated with water, causing the circuit to go down, until we just wait for it to dry out because nothing is permamently destroyed so why would the ISP shell out the money to bury an entirely new T1 circuit **in 2021** when it will fix itself.

Heat is the next issue and this goes back to Ohm's law but for the most part less catastrophic. Even when power is perfectly fine and there is no surge or sag causing the current to go crazy, all powered components will generate some form of heat during normal operation. If the ambient heat of the room the device is in is too high, or if it has a heavy coating of dust acting like a winter coat and trapping the heat onto components, resistance will increase. The good thing though is this is a gradual increase in resistance when compared to a lightning strike.

Devices will get a bit sluggish thanks to the slightly higher resistance affecting voltage and current and seen by any number of tech support posts asking "why is my computer slow" when the inside of the case looks like a carpet. Unless that device is baking itself over long periods of time it won't outright fail it'll just have a shorter life span, and most computer related devices have thermistors which can read the temperature based off the resistance and will do a bit of thermal throttling,which is the real main cause of the slow performance, until it's bad enough it will shut itself off.

Lastly we have signal noise. This mostly happens with wireless communication but can still happen with wired communication. An outside force or signal starts interfering with your signal. A microwave oven being turned on will emit enough RF radiation on the same frequency as 2.4GHz WiFi. Modern ones are pretty well insulated to prevent this noise from bleeding outside the microwave itself but some still can get past the shielding and degrade the WiFi signal near it.

On wired connections you can get issues like cross talk. The electrical signals being sent down the wire creates a small magnetic field which can interfere with the electrical signal of the wire next to it almost like sticking a magnet against a TV screen distorts or destroys the image. This isn't something you need to worry about in day to day use with ethernet because every "Category" cable uses one or more twisted pairs and the engineering of the twist with what is paired to what uses the magnetic fields to cancel each other out and stop that interference or cross talk from happening.

### Hardware Limitations

The last thing I want to cover before moving onto Layer 2 is physical limitations of the hardware. These are all going to be based on the engineering aspects we've covered so far. Yes, it wasn't all just tangents.

#### Cable Length and Attenuation

Attenuation is the increased loss through the medium. Just like WiFi signals getting weaker as you move away from an access point, the farther a wired signal has to travel it gets weaker. This comes from a mix of the resistance the signal is subjected to across the copper wire, a longer wire is higher resistance, and any external factors like the electromagnetism that twisted pair wires tries to negate.

I say "tries to negate" because it's not perfect. Both Category 5 and 5e cables have a maximum distance of 100 meters before all bets are off on the signal, in practice 90 meters is the usual stopping point to make sure there's some extra slack for patch panels and things like that.

The difference between the two is CAT5 can only push 100Mbps at that distance but CAT5e can push 1Gbps at that distance. The physical reason for that difference in speed is the twist rate of the pairs. 1Gbps is a higher frequency so it generates more electromagnetism. As a result CAT5e was designed to have it's pairs be twisted much tighter to reduce that electromagnetism, thus reducing crosstalk.

#### Backplane Speeds

The NIC and the cable aren't the only place where the speed in which data is transmitted is limited. Each device has a maximum bit throughput it's internal components can receive as input, process whatever logic the data abstracted from those bits calls for, and then output the resulting bits.

For example lets look at Cisco 2900 series routers. Technically they all come 2-3 onboard 1000Base-T NICs and the ability to slot in additional NICs in the form of removeable cards. Looking at a 2911 and seeing it's three 1Gbps interfaces you may think it can do at minimum 1Gbps of throughput, so bits moving in being processed and moving out, but that's not the case.

You need to pull up a data sheet for that particular model and look for what the advertised backplane or max throughput is. The components and how they are used will limit speeds internally to the device regardless of the standard used for it's interfaces and sometimes it's hard to find the actual rated speed for the device and not some sales pitch for the entire series of devices. When you finally find it, you then get to come to terms a Cisco 2911 can only handle about 180Mbps **total** through it's three onboard 1Gbps interfaces. If you're running any high CPU features on that device like firewall functions that max throughput goes down.

You can turn features off and increase the max throughput but you'll never go above the theoretical max throughput of it's backplane, and that 180Mbps is in *perfect* conditions. Inevitably someone will complain that it's a 1Gbps connection so it should be 1Gbps.. but the only way to get 1Gbps is to ditch the 2911 that was picked up for a few hundred dollars off ebay or a couple grand from a VAR and buy at minimum a 4431 that has to have all of it's features turned off to get to that speed for a whopping $8-10k or safely run 1Gbps with whatever needed features with a roughly $16-20k 4451.

#### Inherent Latency from Nodes and Distance

The last thing to take into account is latency. The sheer number of issues I've had to "troubleshoot" in regards to latency, sometimes the complaint being slow speeds or bandwidth but it's actually from latency, that are unfixable because there is no actual issue to fix makes me want to pull my hair out and yell "YOU CAN'T BREAK PHYSICS". No form of movement is instantaneous. The speed of light in a vacuum is the fastest anything can be, and it's not instant it's roughly 299,792,458 meters per second. That means anything that isn't light has to be slower, and anything not in a vacuum is also slower.

When it comes to a CAT5e cable, the electrons move through the copper at 2/3rds the speed of light. Taking a 100 meter CAT5e cable, since that's it's max distance before the signal is lost, you at bare minimum have 520 nanoseconds of latency for the signal to start at one end and reach the other end. This is without taking into account any time the devices on either side of that 100 meters cable need to process whatever data before transmitting each bit,

If the distance to travel is 300 meters you need about 4 routers or switches minimum which will generate a new signal before the signal disappears at each 100 meter mark. The actual time it takes for each router or switch or whatever device to take the signal and process it and generate a new signal for the next leg of the trip of course varies based on what the device actually is and what it's doing.

In best conditions, a switch that doesn't have to do any kind of lookups should be able to process that signal and retransmit it in 20-30 microseconds but if lookups are needed all bets are off. Just as a rule of thumb assume each device in the path will can add anywhere from 1 to 10ms of latency.

Fiber partially solves that problem because it can go much farther than copper before signal loss. When you need to send the signal a long distance fiber will come out on top from eliminating the number of devices needed in the path thus removing more of the large jumps in latency those devices cause. **But it still has a limit to it's distance**, and light moving through fiber is slower than electrons moving through copper.

I'm trying to drive this home because one issue I worked will forever be burned into my mind. It was a "latency issue" with tunneled SMB traffic between Pemba and London. Just a straight shot between Mozambique and London is around 8000km, so if we guesstimate the latency of fiber at 5 microseconds per kilometer and take everything else out of the equation and ignore any real world conditions and max distance between devices it's 40ms of latency. They were complaining about 100ms which almost sounded impossible with that straight line drawn on the map once I tried to factor in devices in that path.

After giving that rough explanation that hey 100ms (ROUND TRIP TIME!!! SO ITS THERE AND BACK!!!) wasn't good enough and I needed to solve the latency issue. Well it was time to bust out as much OSINT as I could muster. I ended up finding the fiber rollout plans for Mozambique from a few years prior which gave me the different actual paths this traffic could be taking, most likely dumping into Johannesburg as there was only 2 or 3 fiber connections between Mozambique and neighboring countries. That can't be it, the straight line from Joburg to London is longer and now we exceed 100ms of theoretical round trip time with our unlimited distance fiber.

Alright where else on this fiber rollout map could it be going. Probably to one of the main interconnections in a major city or port like Nacala or Maputo. Time to traceroute to find the path, do some lookups for the IPs in the path against who is registered to own them. Some small ISP, cool. What about using the BGP looking glass and see where they're advertised from?

Turns out it's a company that deals with undersea cables and eventually I got it narrowed down to an undersea cable from Maputo that goes up the north eastern coast of Africa and across the Mediterranean to France, and then France to the UK. Doing the math the 50ms from Pemba to London and 50ms return trip from London to Pemba now makes some sense. Thank you Google search parameters for helping me find all the info on where the fiber is located and showing me who has undersea cables where.

I laid all of this out and the customer finally accepted they **can't break physiscs**. This is as good as they're going to get.

Unfortunately, I have tons of stories like this that all boil down to physics doesn't allow what you want. I would love for more people to understand why so I don't have to add any more stories, and if you read all of this I want to thank you for being exposed to these unsolvable problems.

# Layer 2 – Data-Link Layer In Depth

Let’s start off with the first layer where decisions on where to send data is made. Switches operate here and make all their forwarding decisions based on the physical address of network interfaces, their MAC addresses.

## MAC Addresses

At the most basic level, MAC addresses are the physical address burned into a physical interface on a Network Interface Card, NIC, by the manufacturer. IEEE keeps a registry of what blocks they've assigned to a manufacturer to prevent overlap. In theory, without MAC spoofing, every interface will have a unique MAC address derived from the manufacturer's Organizational Unique Identifier, OUI, then the rest being a sort of unique serial number as that manufacturer produces more NICs and assigns them whatever numbers haven't been previously assigned.

The MAC address itself is 24 bits in length, formatted to be human readable in 6 hexadecimal octets, for example `AA-BB-CC-DD-EE-FF.` The first three octets, `AA-BB-CC` from our example, are the OUI and if you ever used a MAC lookup/OUI lookup such as Wireshark's this is why you can tell if it is a Dell device, Cisco device, etc. The last three octets will be a unique string to that particular interface, be it a physical ethernet interface or that specific radio on a wireless interface. This is the physical Layer 2 Address per the OSI Model.

As a side note and what's helped me in the past in an environment with next to no documentation, NICs with multiple interfaces will <em>generally</em> have MAC addresses in a numeric sequence. If our example of `AA-BB-CC-DD-EE-FF` is a 4-port router you shouldn't be surprised if the other ports are `AA-BB-CC-DD-EF-00`, `AA-BB-CC-DD-EF-01`, and so on.

## Switching

As we established, switches operate at Layer 2 but what are they <em>really</em> besides my way to get more ports in my home lab to connect the ridiculous amount of Raspberry Pis I’ve acquired?

In the [Layer 2 OSI Model section](#layer-2---data-link) I touched on the data itself being in the format of an Ethernet Frame. Using Wikipedia’s image as I am not an artist, a frame looks like this:

![layer 2 frame](https://upload.wikimedia.org/wikipedia/commons/1/13/Ethernet_Type_II_Frame_format.svg)

<cite> Above diagram sourced from https://en.wikipedia.org/wiki/Ethernet_frame#Ethernet_II </cite>

I’ll go deeper into headers themselves more specifically later, but you can see that there is the block for data/payload, the CRC trailer, and important for our use now the MAC header that contains the Source and Destination MAC addresses.

When your device sends data out to the network it will create the layer 2 encapsulation, constructing those frame headers, and write it’s own MAC as the Source MAC Address and then write the physical address for the next hop across the wire from your device’s NIC as the Destination MAC Address. If your device is directly connected to what ultimately is getting the data, then that device on the far side sees it’s own MAC address in the Destination MAC field and will de-encapsulate up the layers until it gets to the pertinent data you sent.

But what if we are not directly connected? What if the reason we’re not directly connected is because there’s a dozen, or several dozen, devices that need to talk on the same layer 2 network?

Enter: Switches

The purpose of a switch is to expand your layer 2 network, and by doing this expand the broadcast domain of that local area network.  To accomplish this a switch maintains it’s own Content Addressable Memory Table, the CAM Table, which is it’s own database so to speak of what interfaces it’s seen what specific MAC addresses send traffic on.

The size of the layer 2 network depends on when you start performing routing at layer 3, so it can be as small as a single cable between two routers or as large as a VPLS built on top of an MPLS network that connects multiple physical sites across one or more service providers enabling a logical topology that looks like just having a really long wire between all the site's respective backbone switches when physically it's ISP circuits that connect to carrier aggregate network routers and switches going through patch panels and long haul fiber between cities, countries, or however complicated it needs to be based on the geographic coverage.

All of this aside-- from your perspective, your sites are just plugging into magical carrier switch, made just for you, bridging the distance between them all as if you are directly connected.

You can see this illustrated a little better in the following diagram, each red box shows the full area covered by each individual layer 2 domain.

![layer 2 domain sizes](assets/images/layer-2-domain-sizes.png)

<cite> Topology made with diagrams.net (formerly draw.io) </cite>

Say you just freshly booted up that switch and plugged your computer into port 1 of the switch. The very first frame your computer sends to that switch, be it from a DHCP request or just sending an ICMP echo to literally any IP address, the switch will see the frame coming from port 1 with the Source MAC Address your computer wrote onto that frame header based on it’s own NIC and record your MAC as being reachable on port 1.

That record will stay in the switch’s CAM table for as long as that switch is powered on as well as the switch’s configured aging timer to remove entries that haven’t been verified (seen any traffic to prove that MAC is still on that port), so as long as you haven’t cleared the volatile memory that table is saved to on the switch and have been regularly sending traffic the switch will know your PC is on port 1.

If the device you’re ultimately sending traffic to has done the same and your computer knows that device’s MAC address, the decision making is rather simple. Your computer writes the Destination MAC address of the other device into the frame header when it builds the frame and sends it across the wire to the switch and the switch sees MAC foo on port 1 is trying to send this frame to MAC bar on port 2 per my CAM table, and switches that frame to port 2 for the other device to receive and de-encapsulate to get the juicy data you sent.

Generally the switch will use it’s physical ASIC chips to make these forwarding decisions to switch the frames at high speeds, this is why theoretically you’ll get better throughput on data transfers using a switch rather than having a router connect your computer to another device, as routers/layer 3 devices will generally use software instead of hardware to make these decisions.. but that’s getting ahead of ourselves here.

What if the other device connected to the switch has been idle for the past 5, 10, 30 minutes and it’s MAC address isn’t in the switch’s CAM table? Then the switch will broadcast ARP or Neighbor Discovery frames out of every interface that currently has something plugged into it.

When I said switches increase the broadcast domain of a network, this is a key aspect for that. We’ll need to go over Layer 3 before talking about ARP or Neighbor Discovery but for now we can summarize it and broadcasts in general as one-to-all messages. On layer 2 the destination MAC address will be written as `FF:FF:FF:FF:FF:FF` for broadcasts, basically this means “everyone who hears this message should read it”.

How this helps in the above scenario is the switch will see the destination MAC of the device your PC wants to send a frame to, but the switch currently doesn’t know where to send that. The switch then broadcasts to everything connected to it asking “Who is this Destination MAC Address?” so when the device off port 2 receives that it first reads the frame because the all F’s destination forces it to process that frame instead of dropping it as a message not meant for it, then goes “Hey that’s me” and tells the switch, thus loading that entry into the CAM table. Once this is done the frame your PC sent to the switch, after waiting for the verification of where that destination currently is accessible from, will be sent from the switch to the other device.

### Layer 2 Segmentation with VLANs

We know how one or more switches will increase the Layer 2 Domain (Broadcast Domain), now for the opposite, segmenting the large broadcast domain into multiple smaller domains.

Virtual Local Area Networks, or VLANs, are essentially just a way to logically break up a physical layer 2 switch into multiple logical switches. Once a VLAN is configured on a switch and that VLAN is then assigned to different interfaces on a switch, it will function like those interfaces belong to a completely different switch even though it’s the same physical hardware. For instance, if you have 10 ports and build then assign the first 5 to VLAN 10 and the last 5 to VLAN 20 then all the functions and characteristics of a Switch described in the above paragraphs will be unique to that VLAN.

Broadcasted traffic will only go to other interfaces that belong in the same VLAN, the CAM table entries (depending on how that vendor does their syntax) will reflect MAC addresses what VLAN they belong to as reflected by the interface’s assigned VLAN. Additionally, now that they are logically separated as if they are two different switches and there is no cable plugged into an interface on VLAN 10 that then goes to an interface on VLAN 20, which please never do for the sake of the network engineer’s sanity and defeats the purpose of VLANs, the two VLANs cannot interact with each other unless routing is performed somewhere that will let the two VLANs talk to each other.

The way this is accomplished, configuration being ignored, is the switch will add onto the Layer 2 frame to add an 802.1Q header, also called the VLAN tag, as seen below:

![dot1q Frame](https://upload.wikimedia.org/wikipedia/commons/0/0e/Ethernet_802.1Q_Insert.svg)

<cite> Above Diagram sourced from https://en.wikipedia.org/wiki/IEEE_802.1Q#Frame_format </cite>

Inside this header there is a 12-bit field for the VLAN ID, allowing the switch to slap a number between 0 and 4096 on a received frame from any interface that was assigned to a specific VLAN.

Worth noting is that this header is only applied to trunk ports-- interfaces on the switch that are used to trunk multiple VLANs across multiple switches or to differentiate what frames are from what VLANs when the switch is connected to a router that will perform Inter-VLAN routing while treating that single cable with multiple VLANs piped into it as individual routed interfaces, which will be more in depth in the Layer 3 portion after we explain how a router functions.

All other interfaces on the switch that go to different hosts on the network, PCs, VoIP phones, servers, and so on, are access ports.

When a frame arrives from a PC on an interface assigned to VLAN 10, it does not at that moment get the dot1Q header inserted into the frame. The switch will take a note that it’s a frame from that access port and only switch it out other interfaces that belong to the same VLAN. If that frame is going to another access port locally to the same switch in that same VLAN it will just switch the frame without modifying it because it knows it’s own interfaces and how they are configured.

**If the destination is not local to the same switch, the switch will only insert the dot1Q header into the frame when it is being generated to egress one of it's trunk ports to go to a neighboring switch. The neighboring switch that receives the tagged frame on it's ingress port will strip the tag off when checking if that VLAN ID is allowed on itself.**

When that neighboring switch removes the VLAN tag it makes a note of the ID so that it will continue to keep that frame isolated to the right VLAN and either switch the frame to a destination that's on a local access port or see in the CAM table for that VLAN the destination address is learned from another trunk port that goes to another switch. At that point when the frame is being re-assembled on it’s local trunk port to be sent across it will add the 802.1Q header again to re-add the VLAN tag for the next switch’s benefit. This process repeats for every hop until the layer 2 destination is finally reached.

The reason I’m getting this in depth on VLANs is due to how they can seriously impact the flow of traffic and reachability of devices in the network. This next diagram will visualize why I find this so important.

![layer 2 network](assets/images/layer-2-network.png)

<cite> Topology made with diagrams.net (formerly draw.io) </cite>

Here we have a rather small and simple network topology with the following nodes:

- The ISP’s edge device, either a customer premise device like a fiber ONT, T1 smartjack, some form of service access switch or if the ISP has no equipment besides the circuit itself ran from a pole--  then the node is the provider edge device in a Central Office or similar point of presence.
- The Router used by this site to connect all internal VLANs as well as function as the WAN connection to give internal devices access to the Internet.
- The Support Desk worker’s PC which does not actually know it is plugged into an access port for VLAN 10 but just knows it has layer 2 reachability to the ticketing system and the Router.
- The IT Admin’s PC which does not actually know it is plugged into an access port for VLAN 20 but just knows it has layer 2 reachability to the server for management and the Router.
- Switch 1 that directly connects the Support Desk worker’s PC and Switch 2 with VLAN 10 access ports, Switch 3 with a VLAN 20 access port, and a trunk port that allows both VLAN 10 and 20 have connectivity to the Router.
- Switch 2 that directly connects Switch 1 as well as the interface on the Server that is setup to allow access to the ticketing system webpage with VLAN 10 access ports.
- Switch 3 that directly connects Switch 1, the IT Admin’s PC, and the interface setup to give management access to the Server as VLAN 20 access ports.

To start off, suppose the Router is not setup to allow the two networks, VLAN 10 and 20, communicate with each other but they both are able to reach the Internet. This means the Support Desk PC is only able to reach what is explicitly within VLAN 10. It cannot reach the Server’s management interface, the IT Admin’s PC, and even Switch 3 will not receive a single frame that the Support Desk PC has ever sent even if it is broadcast traffic.

On the other hand, the IT Admin PC cannot reach the ticketing system’s web page even though it has reachability to the Server’s management interface as well no broadcast traffic it sends will reach Switch 2 <em>and never reach the Support Desk PC</em> even though VLAN 20 exists on the same switch the Support Desk PC is directly connected to, the IT Admin PC’s broadcasted frames are localized <em>only</em> to VLAN 20.

If this is how the network is designed to function, great, but we don’t live in a perfect world where a network will stay the way it is designed for all time or that the person designing the network didn’t make a mistake or overlook a detail.

What if someone swapped the cable connecting Switch 1 to Switch 2 with the cable that connects Switch 1 to Switch 3? Now the Support Desk worker can’t access the ticketing system but can access server management and vice versa for the IT Admin.

What if one of the cables connected to the Server fails? You’ve lost management or ticketing system access even though there is still one functional cable attached to the server.

### Issues From Lack of Layer 2 Segmentation: Loops

Another thing to consider is layer 2 loops. This is less about security and more about network stability.

![layer 2 loops](assets/images/layer-2-loops.png)

The connections in the above image show three general loop inducing scenarios. Circling back to broadcast traffic, when a switch receives a frame destined for `FF:FF:FF:FF:FF:FF` it will forward that frame to every single interface except the interface it had received it on. In all three of the above scenarios you can see how that can create a never ending loop as the broadcast frames just spin around the network in a circle in what's called a broadcast storm.

This isn't limited to just broadcast traffic either. If there is a PC attached to one of the switches in the four-switch scenario, as the CAM tables of the other switches are populated with which MAC address is reachable via which interface (as inferred by seeing the source MAC address on every frame received ingressing a specific interface) they will eventually see that MAC address reachable by the interface connected to the switch clockwise to it and the interface connected to the switch counter clockwise to it.

The CAM table will eventually become unstable which will decrease the switch's performance and duplicate frames will be created since the destination MAC address is reachable on two egress interfaces which will cause issues when those frames are transporting something like IPSEC traffic which will cause the device receiving the duplicate IPSEC traffic to think there's a replay attack.

The way you stop this without giving up the redundancy of multiple paths is through some form of spanning tree protocol, [which wikipedia gives a good breakdown of what STP is and the different forms it can take like per VLAN spanning tree or rapid spanning tree.](https://en.wikipedia.org/wiki/Spanning_Tree_Protocol)

STP will do some decision making based on the which switch is elected as the root bridge, basically the switch in charge of the STP instance that is either manually set or based on the MAC address which will turn out to be the oldest switch, and will tell the switch farthest away from it to disable all but one of the redundant connections, as portrayed by the prohibited symbols in the above scenarios. This will break the loop and if the active interface were to go down if the cable breaks or that neighboring switch fails, that far end switch will re-enable a deactivated interface automatically to give an automated failover.

### Native VLANs

Before moving on from VLANs there is one more thing I wanted to cover, Native VLANs. I intentionally waited until after the different aspects of segmenting layer 2 were covered so that the full impact and considerations for the Native VLAN made sense.

When you configure a switchport to be a trunk you add different VLANs to be able to traverse across that trunk to reach the neighboring switch. These VLANs can exist on the switch you are configuring, like switchports gig 0/2 and 0/3 are setup as access VLAN 10. They could also exist on *different* switches on your LAN so that the VLAN tag is preserved when traversing from Switch 1 to Switch 2 to Switch 3.

If VLAN 10 is only applied to access ports on Switch 1 and 3 but *not* Switch 2, you still need to add VLAN 10 on the trunk ports on Switch 2 to preserve the segmentation. This lets a frame tagged with VLAN 10 as it egresses Switch 1 to be received by Switch 2's end of the first trunk without being discarded as well as being re-tagged **by Switch 2** when the frame is then egressing from it towards Switch 3 across the second trunk.

Now, the issue that can happen is what if Switch 2 doesn't support VLAN tagging? Let's say Switches 1 and 3 are your bigger 52-port switches but Switch 2 is acting like a bandaid fix to overcome the physical distance limitation of CAT5e by acting as a signal repeater while also being convenient for the two PCs that are closer to Switch 2 than the other switches. If VLAN 10 is used for PCs but we're back in the late 90s/early 2000's when 802.1Q wasn't fully supported by every device, how do we make sure that we have proper segmentation on Switches 1 and 3 for VLANs 10, 20, and 30 without kneecapping the two PCs on Switch 2 that need to be on VLAN 10?

That's where Native VLANs come in. A Native VLAN is an **untagged VLAN**, even if it goes across a trunk port. On Switches 1 and 3 you'll configure their respective switchports that connect to Switch 2 as trunks with VLANs 10, 20, and 30 like normal **with the additional step of setting VLAN 10 as the native VLAN.** This makes it so frames that originate from access ports on VLANs 20 and 30 on those two switches will get tagged with their respective VLAN IDs, yet frames that are coming from the PC access ports on VLAN 10 **do not** get a VLAN tag.

For example, when broadcast frames from VLANs 20 and 30 cross the trunk ports from Switches 1 or 3 and ingress into Switch 2, they will be broadcasted out *every* active switchport-- including the switchports that the two PCs are connected to. Those PCs receive these frames that have 802.1Q headers and then go "what the hell is this?!" and drop those tagged frames, keeping traffic that should only be destined to by devices in VLANs 20 and 30 from being processed by those two PCs on Switch 2.

On the flip side, because VLAN 10 is the native VLAN on switches 1 and 3, frames broadcasted by a PC on Switch 1 **do not** get tagged as it crosses the trunk into Switch 2. Those frames get broadcasted out every switchport like last time but now those two PCs on Switch 2 are receiving frames *without* the 802.1Q header and the PCs will process them as we intended. When Switch 3 receives the untagged broadcast frames on its trunk port from Switch 2, and because VLAN 10 is the Native VLAN, it assumes that these untagged frames belong in VLAN 10 and will forward the frames to every access port in VLAN 10 so that the PCs connected to Switch 3 can receive and process the frame.

The last thing about Native VLANs will be a perfect segue into the next section, what happens when you mess up the Native VLAN config between switches. Let's go back to two switches trunked together for our example, Switch 1 and 2. Both switches support 802.1Q and are setup to trunk VLAN 10 and VLAN 20 and we want to make VLAN 10 our Native VLAN. We'll keep PCs on VLAN 10 for consistency and put our hypothetical servers in VLAN 20

Here's the twist: VLAN 1 is the Native VLAN on Cisco switches as well as the other vendors that copy Cisco's defaults.. And you misread Switch 2's output that stated the Native VLAN is 1 by thinking it said VLAN 10.

Remember how the Native VLAN is the untagged VLAN? Here's what happens. When either switch receives frames from the servers on VLAN 20 that need to go to the *other* switch, those frames are sent across the trunk with the 802.1Q header listing VLAN 20. That tag gets applied as the frame egresses the trunk port and is then stripped when it first ingresses the receiving switch's trunk port on the other side. When either switch receives frames *from the PCs on VLAN 10* then the misconfiguration gets complicated.

Since Switch 1 has the correct Native VLAN of 10, any **un-tagged** frames that are received on any of it's trunk ports are assumed to belong in VLAN 10. If Switch 2 really does have VLAN 10 and 20 configured on its trunk port then the frames sent by PCs on that switch will still traverse the trunk but have an 802.1Q tag stating VLAN 10. Switch 1 receives that... then finds it kind of odd that those frames were tagged with VLAN 10 but forwards them regardless to the correct destinations that belong in VLAN 10.

Conversely, when Switch 1 sends frames destined to PCs on Switch 2.. those frames don't have VLAN tags, VLAN 10 is the Native VLAN on Switch 1 still. When those *untagged* frames ingress Switch 2 and get processed, and because they're untagged, Switch 2 then assumes they're frames that belong to VLAN 1. If there is no inter-VLAN routing (performed at Layer 3), then the PCs on Switch 2 would be able to talk to the PCs on Switch 1 but they would **never** see the response.

The flow of traffic turns into this:

1. The broadcast frame originally sent by a PC on Switch 2 egresses the PC
2. Switch 2 receives that frame, sees that it is a broadcast frame and would need to go to Switch 1
3. Switch 2 forwards the VLAN 10 tagged broadcast frame to Switch 1
4. Switch 1 sees the VLAN 10 tag, generates a notification-level error message that there is a Native VLAN mismatch
5. Switch 1 forwards the broadcast frame to all VLAN 10 related switchports, allowing the PCs to receive it
6. The PCs then reply, sending frames containing the response to the broadcast to Switch 1
7. Switch 1 sees it needs to go to the MAC address of the PC in step 1 which it learned on VLAN 10 of the trunk
8. Switch 1 forwards the **untagged** responses to Switch 2
9. Switch 2 sees these untagged frames **and thinks they belong in VLAN 1**
10. Those responses are then discarded because Switch 2 has no switchports that belong in VLAN 1

That example is more focused on lack of connectivity, or rather one way connectivity, but in certain scenarios this can lead to other unintended behaviors.

What if VLAN 20 was the Native VLAN on Switch 2? What if those servers on VLAN 20 handle credit card data? How would your PCI-DSS audit go?

## Layer 2 Security and Network Stability Concerns

Besides breaking things these simple changes or failures can impact security. An issue I’ve run into a lot while working as a VoIP provider with an onsite network stack for the VoIP phones is when a customer replaces a switch or overhauls their network stack and introduces similar issues.

When you have a blended network at a customer site with routers and switches on the MSP side as well as routers and switches on the customer side, to save some headache of having to do two cable runs for every employee desk we would have the drops run through our switches, tag the phone traffic on our gear and tag the data traffic that would come from desktops daisy chained in the PC port of the phone to dump into the handoff for the customer’s switches to reach their network environment.

Only one ethernet run would be needed for each desk to simplify internal wiring, the voice traffic for the phones stays on our network for protection and quality of service, and the PC traffic is separate from our management and voice network with the only egress point being the customer network so they can still reach customer Active Directory, DHCP and Web Servers, and follow whatever rules the customer’s firewall is setup for internet access.

Whenever the customer swaps equipment or changes their wiring without keeping the interconnect between the two networks the same as during initial install, security and access issues pop up fast. Either a customer switch will now connect to a port on our managed switch that isn’t setup for the data VLAN to dump to them which would break all the PCs from having network access to anything that isn’t another PC on our switches’ data VLAN… or worse… the ethernet runs or the patch panel will be changed and phones will now be on the customer network and either completely break or if a certain sequence of events happen the phones will function but the customer’s router or firewall will not be protecting them and within hours they’ll be compromised by SIPVicious.

 This is why I hammer home how layer 2 segmentation works and tell others to be cognizant of the human element, mistakes happen and we need to be aware of them if something acts in a way we don’t expect.

## Layer 2 Traffic Types

I want to make it clear at this point what the different types of traffic in layer 2 are after referencing broadcast traffic a handful of times. Traffic types are going to be brought up a few more times in the higher layers. While they first come up in layer 2, understanding the theory and application of each type of traffic will remain a core concept from here on out.

I'll relate this back to human verbal communication to give a foundation on these different types of traffic so we have solid ground to build our understanding on, but there's something that needs to be explained first. If I haven't clearly explained what "traffic" means yet you can think of it like road traffic where it's a grouping of vehicles or pedestrians that are in the state of moving through a space.

To pull this back to the context of networking, swap out roads or footpaths in the real world with physical layer media and the moving vehicles or pedestrians with that particular OSI layer's unit of data, in layer 2's case they will be frames.

However, our working example in the different types of traffic is going to be based on verbal communication, so think of the flow of spoken words in a conversation as the flow of vehicle traffic on a road. Clear as mud? Cool.

Imagine you're in a meeting with your team, the networking team, but there are other teams present in the meeting as well like the desktop support team and the systems administration team-- all in one meeting room. Joining you on your team is Bob and Alice.

If you want to say something to the entire meeting you might start out by saying "Everyone," or some variation of that so all the teams physically present in the room know you're addressing them. This would be a *broadcast* message, because you're not limiting the extent of who should be listening to one person or one team.

If you wanted to say something directly to Bob you may start out looking directly at Bob then saying "hey Bob," to let Bob know you're addressing him and not Alice or anyone in a different team. This directed communication would be a *unicast* message. Sure other people may hear you but you started out specifically addressing Bob so they can disregard anything else you say, which would be the case in a computer network when a WiFi Access Point is "speaking" or if there is a layer 1 hub that is replicating the frame out every interface since hubs act as repeaters and aren't capable in understanding what device or MAC address is on the other end of it's interface.

Lastly, you can address the entire networking team at once without having to address Bob specifically then re-say what you told Bob a second time when addressing Alice specifically by starting your statement with something like "networking team,". You only need to speak once but everyone on the networking team, Bob and Alice, would be paying attention while the desktop and sysadmin teams know it's not something directed at them.

That means you can do the same thing for those other teams by addressing desktop support or systems administration so the respective team being addressed in your statement will pay attention and the others can safely ignore what you're saying. This is a *multicast* message.

Therefore the different types of layer 2 traffic are:

- Unicast Frame - One-to-One layer 2 message directed at a specific MAC address.
- Broadcast Frame - One-to-All layer 2 message directed at the broadcast address FF-FF-FF-FF-FF-FF.
- Multicast Frame - One-to-Many layer 2 message directed at the multicast address that depends on the higher layer protocol.

Notice how I said the layer 2 multicast address depends on a higher layer protocol. Take a second and think about the meeting scenario again but try to place what was involved into different layers of the OSI model. The open air of the room is the layer 1 physical medium. It doesn't care about what's being said or by whom, it only acts as the space your words travel through. Each person's mouth and ears are the layer 1 interface which just do the act of sending or receiving the spoken words.

This is probably a leap to work in analogy wise but that metaphysical or psychological gap between your thoughts and actually speaking them in a structured manner with your mouth, along with the reverse in the gap between your ears hearing those sounds and abstracting them into ideas would be layer 2.

If we change this up a little bit and say no one speaks a common language in the room, you can at least pick up on the social cues when someone is looking directly at you while speaking to know you're being addressed to cover the unicast idea. Broadcast wise, I'm assuming everyone reading this has had to give a presentation at school and the teacher told you to not speak directly at one person so you try to look in every general direction but not directly at a person, scanning back and forth, to give off the idea you're not speaking to just one person but to everyone.

Then there's multicast. How do you address a certain group of people if you can't rely on those instinctual social cues since Bob may be on the far left of the room and Alice may be on the far right? At some point your face is going to glance at someone that isn't on your team. Bob and Alice being on the networking team *and knowing they're on the networking team* takes some understanding of the language you're speaking, an understanding of a team on a conceptual level, recognition that they belong to that conceptual "team", and however deep you want to go into these philosophical questions.

Therefore, multicast isn't a fully fleshed type of traffic in layer 2 but it is still involved in multicast traffic because higher layer messages and their actual contents are going to be encapsulated at layer 2 into a frame. This would be the  same as the English words "networking team" along with the contextual understanding of "networking team" are encapsulated by the structured sounds you are speaking, and conversely, your ears de-encapsulate the structured sounds so your brain can interpret their meanings.

So if unicast traffic at layer 2 has a destination address of a NIC's MAC address, broadcast traffic at layer 2 has a destination address of all F's, and specific multicast groups or addresses aren't known to layer 2 since they're a higher layer function,where does the destination MAC address come from? How does a switch forward multicast frames? **How the hell would multicast even work at layer 2?!**

Thankfully the world uses almost solely Ethernet-based networks, so this gets answered in IEEE's 802 standards as opposed to different networking equipment vendors coming up with their own solution.

Let's take care of the address part first. If you've been reading between the lines so far you'll see there can't be one universal multicast address or a group of devices wouldn't be able to determine a multicast frame is for them or a completely different group of devices also using multicast.

There's also some unifying purpose for multiple devices to be grouped in the first place which would require the various multicast address to be universally understood, just not by layer 2. As a result, IEEE has standardized a single layer 2 address of FF-FF-FF-FF-FF-FF to be used for specifically broadcast traffic and the remaining 2 to the power of 48 addresses be equally divided between unicast and multicast, 2^48 because a MAC is in binary and 48 bits long.

The division of the remaining 281.5 *trillion* potential MAC addresses is pretty simple. Look at the most significant byte which would be first/left-most byte and see if it's most significant bit is a 1 or a 0. If we take an example MAC address off AA-BB-CC-DD-EE-FF, the most significant byte is 0xAA, which is 10101010 when converted into binary. The most significant bit inside that byte is a 1, so per IEEE it will be considered a multicast address. If the most significant byte was 0x64, 01100100 in binary, then it's unicast and would belong to a specific interface.

When a switch receives a frame and the most significant bit in the most significant byte is a 1, it treats it like a broadcast frame and forwards the frame out every interface except for where the switch first received it. The multicast frame would then be received by every device in the layer 2 network and then each individual device would let the higher layers determine if it needs to process the contents of the frame and act on them.

This may sound counter-intuitive but I'm going to wait until Layer 3 to explain multicast addresses any deeper. Within the scope of layer 2 you just need to consider the very first bit in the destination MAC address, and not the remaining 47 bits.

# TCP/IP (DOD) Model's Link Layer: Yes I Lied About Skipping This

## Why I'm Covering It Now

The dilemma I have writing this lies with how the both models were developed and their strengths and weaknesses.

The OSI model was developed by committee across the entire industry to make a common network standard everyone can use to try and assure interoperability between each vendor. As a result, it's not exactly a perfect representation of TCP/IP networking because it really focused on being a framework for all forms of networking, including old telecommunication networks.

The TCP/IP Model that came out of DARPA focuses specifically on the internet protocol suite, which technically is the proper name of the model. Things that seem loosely connected in the OSI model, when we're talking about modern computer networks that connect on the lower level via Ethernet and connect on the higher level with IP addresses and ports, are clear cut in the DoD model because it was made *for this network implementation*.

From a holistic point of view I find the OSI model to be the best way to understand the basic concepts and get a universal understanding of the concepts. I personally started out dual majoring in college in Computer Network Systems and Computer System Support Technology where the networking specific classes in that first major were centered around Cisco Networking Academy's CCNA and CCNP Routing and Switching courses along with the CCNA Security course.

While we talked about both and the material focused on the OSI model, in practice almost all of the material was based on the TCP/IP suite. The heavy focus on the OSI model helped me tremendously after college when I started working in a role that was equal parts telecommunications provider, internet service provider, and managed services. I went from a self confident CCNP with a solid understanding of enterprise networking to "oh my god, carrier networking is barely the same as enterprise networking.

This is like being a native Italian speaker and learning French" and I certainly didn't know how the VoIP and PSTN worked but now I'm responsible for inter-carrier exchanges too? That's where the OSI model's tech-agnosticism helped get me up to speed quickly and turned me into my department's subject matter expert on ISP and telco work. **But TCP/IP still gives a  much better technical model and understanding when looking purely at modern computer networks.**

### Francis Ford Coppola Presents  IEEE 802: Redux

So far I've also been loosely talking about IEEE 802 working groups like VLANs in 802.1q or loosely referencing WiFi here and there which is 802.11. The majority of the real in depth explanations of IEEE 802.3 I've given so far were within the Layer 1 section and there was still a decent amount in the Layer 2 section, but I never got really deep into yet. There's a reason for my lack of depth about Ethernet and why all flavors of 802 have seemingly popped up all over both sections.

I've held off covering some crucial details about Layers 1 and 2 because I believe its easier to understand them after learning what I have covered. There is a reason for this, everything within the IEEE 802 family exists in both layer 1 *and* layer 2, including ethernet. Since the TCP/IP suite was modeled and developed with both combined there are things that are a varying combination of the two which makes something like the binary preamble of a frame hard to place in just one OSI layer and just hard to understand if you're trying to cover **only** the physical binary signal part of it without knowing what a frame is.

So let's go over everything else like we're adding 49 minutes to Apocalypse Now, making this unbearably long.

## Media Access Control

In the OSI model the Data-Link layer is broken down into two sublayers: Media Access Control, the MAC in MAC address, and the Logical Link Control sublayer. In my opinion the reason the OSI model's second layer ends up being the only layer broken down into sublayers is because the model itself wasn't designed to apply *perfectly* to TCP/IP and IEEE 802, making what should be one whole technology standard be ripped apart into four different layers (counting sublayers).

Media Access Control is where the lines between OSI Layers 1 and 2 blur together. It controls the operation of the physical hardware and media we covered in Layer 1, or more specifically it handles the flow control and processing of what the physical hardware transmits across the specific medium. Between what is written in IEEE 802 and the functions used in practice by Ethernet Media Access Control accomplishes the following:

- The structure of the frame, how to recognize it, how the beginning and ending of a frame is delimited to know the start and stop points.
- The addressing of different "stations", stations being the term for each layer 2 endpoint/device, so the NICs and their MAC addresses
- The actual transmission or movement of Logical Link Control PDUs (frames) in a way that is transparent and readable to higher layers
- Error checking and protection
- Control and access of physical media like speed and duplex or port security

### The Structure of Frames and How to Delimit Them

 We're going to circle back a bit to the Ethernet II frame and dot1q frame diagrams [a few sections back,](#switching) just to be clear when I say frame I'm talking about Ethernet II frames and not IEEE 802.3 frames

In general the size or length of a frame is going to be at minimum 64 bytes and 1518 bytes maximum. Using size vs length really comes down to your perspective though. The bits making up the frame are sent in a linear stream so length would be appropriate but if you are looking at the frame in a packet capture all at once, size I guess sounds more natural.

To complicate this further there is a distinction between the maximum frame size and the Maximum Transmission Unit. The MTU is the maximum size of the data payload from OSI layers 3-7 that are encapsulated within the frame but not including the layer 2 portions of the frame itself. So 48-1500 bytes of payload added to the 18 bytes of layer 2 framing gets 64-1518.

But wait, there's more!

You can then take into account the additional 20 bits used by layer 1 which aren't necessarily part of the actual frame but are still bits that Media Access Control transmits over the wire. So when you start talking about the amount of binary data sent between two layer 2 hops some people can get confused.

Are you talking about the payload after the frame encapsulation is removed? Are you talking about the payload *and* the frame's headers and trailers? Or are you talking about the raw bits sent which includes that completely unrelated, **but absolutely required to work reliably**, extra 20 bits that Media Access Control uses in addition to the last two chunks of bytes?

Can you start to really see why the OSI model blurs lines when TCP/IP and Ethernet are shoehorned into it? For your sanity, dear reader, only an insane person (wait is that me?) means that last one and everyone generally means MTU or the maximum frame size.

Anyway, the maximum frame size/length can be increased above 1538 bytes if you flip Mermaid Man's belt to W for Wumbo and enable jumbo frames on the network devices, which is *kind-of-standardized-but-kind-of-not* in regards to the size of a jumbo frame. Depending on what the vendors are for the devices used in the network path, physical media, and the general design of the whole network the maximum frame size can get to about 9038-9216 bytes. The MTU size when jumbo frames are enabled is 9000 bytes which gives some wiggle room for the additional overhead since 9216 bytes is the maximum size for *most* ethernet devices from a hardware capability standpoint.

The following table is going to be my easy to make frame structure diagram, hopefully covering the total size/length in that last paragraph will make the data payload being a range make a bit more sense. I'm also going to create a separate row to have a distinct example using VLAN tags. The first three frames listed all support VLAN tags, so "Ethernet II Frame" and "Ethernet II+VLAN" are the same exact type of frame but the latter uses the optional VLAN tag.

I hope this isn't confusing, my goal is to illustrate what the frame structure looks like *and draw attention to the VLAN tag's effect on the payload length versus the total frame length.*

| Type              | Preamble | Start Frame Delimiter | Destination MAC | Source MAC | 802.1Q tag | Ethertype/Length | 802.2 Header | Layer 3-7 Payload | Frame Check Sequence | Interpacket Gap |
| ----------------- | -------- | --------------------- | --------------- | ---------- | ---------- | ---------------- | ------------ | ----------------- | -------------------- | --------------- |
| 802.3 Frame       | 7 bytes  | 1 byte                | 6 bytes         | 6 bytes    | DNE        | 2 bytes          | 8 bytes      | 46-1492           | 4 bytes              | 12 bytes        |
| Ethernet II Frame | 7 bytes  | 1 byte                | 6 bytes         | 6 bytes    | DNE        | 2 bytes          | DNE          | 46-1500 bytes     | 4 bytes              | 12 bytes        |
| Jumbo Frame       | 7 bytes  | 1 byte                | 6 bytes         | 6 bytes    | DNE        | 2 bytes          | DNE          | 9000 bytes        | 4 bytes              | 12 bytes        |
| Ethernet 2+VLAN   | 7 bytes  | 1 byte                | 6 bytes         | 6 bytes    | 4 bytes    | 2 bytes          | DNE          | 42-1500 bytes     | 4 bytes              | 12 bytes        |

And for a quick summary table, while also including 802.11 WiFi that is bridged to 100Mbps Ethernet to compare efficiency :

| Type                                  | Layer 1 Overhead        | Layer 2 Overhead | Layer 3 IP Overhead | Layer 4 TCP  Overhead | Max Useable Data Payload | MTU  | Total Frame Size | Total Transmitted Bits    | Useable Payload vs Total Transmitted Bits |
| ------------------------------------- | ----------------------- | ---------------- | ------------------- | --------------------- | ------------------------ | ---- | ---------------- | ------------------------- | ----------------------------------------- |
| 802.3 Frame                           | 20 bytes                | 26 bytes         | 20 bytes            | 20 bytes              | 1452 bytes               | 1500 | 1518 bytes       | 1538 bytes                | ~94%                                      |
| Ethernet II Frame                     | 20 bytes                | 18 bytes         | 20 bytes            | 20 bytes              | 1460 bytes               | 1500 | 1518 bytes       | 1538 bytes                | ~95%                                      |
| Jumbo Frame                           | 20 bytes                | 18 bytes         | 20 bytes            | 20 bytes              | 8960 bytes               | 9000 | 9018 bytes       | 9038 bytes                | ~99%                                      |
| Ethernet II+VLAN                      | 20 bytes                | 22 bytes         | 20 bytes            | 20 bytes              | 1460 bytes               | 1500 | 1522 bytes       | 1544 bytes                | ~94%                                      |
| 802.11 WiFi Frame bridged to Ethernet | 24 bytes + varying DIFS | 56 bytes         | 20 bytes            | 20 bytes              | 1460 bytes               | 1500 | 1580 bytes       | 1580 bytes + varying DIFS | ~92% or way lower                         |

Yeah, it's a lot to take in all at once. Especially with the overlapping/redundant numbers in the second table, since Layer 3 and 4 overhead add up with "data payload" to make MTU which then is added to Layer 2 overhead to make total frame size which then adds Layer 1 overhead for total transmitted bits just to get that single frame across.

Don't stress out, I'll break it down starting with just the actual Layer 2 frame.

### Layer 2 Frame Portion

Focusing on the first table, the frame is everything between Destination MAC and Frame Check Sequence.

In an Ethernet II frame the Destination MAC address, Source MAC address, 802.1Q VLAN tag (if using VLANs), and Ethertype portions all make up the **Layer 2 Frame Header.** Conversely, the Frame Check Sequence, or FCS, makes up the **Layer 2 Frame Trailer.** So what's in front of the payload is the header and what's behind is the trailer and both of those "wrapping" around the payload is what makes this a Layer 2 frame because now the Layer 3 and up data is *encapsulated* with Ethernet II framing.

Most of the header's purpose is for finding out where to go as seen by the Destination and Source MAC address fields [which we covered back in switching](#switching) and the VLAN tag field [back in Layer 2 segmentation.](#layer-2-segmentation-with-vlans) There is one more thing that doesn't have anything to do with where the frame is coming or going and that's the Ethertype field/Length field.

In IEEE 802.3 frames that 2 byte field dictates the length of the entire frame and requires that random 8 bytes I listed as "802.2 Header" to follow it. Ethernet II however uses those 2 bytes for what the frame's Ethertype is and does not include the 8-byte "802.2Header". I'll touch more on both of those in the [logical link control section](#logical-link-control) but I wanted to acknowledge their existence in the first table.

The trailer just consists of the FCS which functions as a cyclic redundancy check. Think of it like a shipment manifest where the entire frame is a large shipment where each field is one of several trucks delivering it's portion of the shipment in linear order as if you only have one loading bay. As each truck is unloaded you make a note of the amount of boxes and what they weigh. At the end of the shipment the last guy hands you the manifest that lists what boxes and weights were dropped off by each of those trucks and you compare the running totals by crossing off the matching weights in both lists. If they match, cool your shipment arrived 100% intact. If they don't match you know something went wrong and either one or all the trucks had their respective cargo tampered with.

So you burn it all.

Dramatic analogy aside, that's the rough idea of what FCS is. The way it actually works is just a lot of algebra and a little boolean logic. The first 32 bits of the frame, which would cover the entire header along with 14-18 bytes of payload (depending on VLAN tag), are complimented (with XOR) and shifted through a polynomial. I'm going to beat a dead horse with this but I'm pretty bad at complex math so I'm just going to link to [this Stack Overflow answer on calculating CRC32 by hand](https://stackoverflow.com/a/4811743) that breaks it down way better than I ever could.

### Layer 1 Frame Portion

Back to what I'm good at, logic and not hard math. The last three portions of framing or encapsulation that the Media Access Control wraps around the payload isn't actually part of the frame. Honestly it has nothing to do with Layer 2 in the OSI Model *even though it's part of that "layer 2 sublayer" since we're going to keep trying to force TCP/IP over Ethernet to work in our special model.*

Now that I'm done screaming into the void we can cover the Ethernet frame preamble, the start frame delimiter, and the interpacket gap. None of these things have any bearing on what to do with the frame or any switching/layer 2 operations, instead they have to do with quite literally controlling the access to the physical media.

To slightly circle back to FCS, think about how it relies on knowing exactly what the first 32 bits of the frame are. That means we need to know what the start of the frame even is to find that first bit. If frames are just being slammed down the wire into the receiving NIC back to back with no break between the last bit of the previous frame's FCS and the first bit of the next frame's destination MAC, a singular skipped beat in this barrage of information can ruin every single frame transmitted after that mistake unless there is a universal way for the transmitting NIC to say "hello" and "goodbye".

Media Access Control solves this problem by using specific coded messages in that 20 bytes of Layer 1 overhead. The first sequence of bits transmitted across the wire when a frame is being sent is the preamble. The preamble itself is a 7 byte pattern that just repeats `10101010` followed by the start frame delimiter is sent which is `10101011`. Think of it like a mix between a jingle playing at the start of a loud speaker announcement that crescendos into the "please leave a message after the beep" voicemail recording tone.

The preamble is supposed to be uniquely useless but also serving as a uniform pattern to make sure both NICs have their bit rate synchronized. After back and forth waveform the voltage double taps high for two 1 bits in a row in the start frame delimiter and the receiving NIC will now know every bit it receives after that is part of a frame.

Once the full layer 2 frame is transmitted, regardless of the success during CRC, there is a 12 byte interpacket gap. IPG is nothing, it's a silent cooldown period. The interpacket gap is an intentional idle period where the voltage isn't high enough to trigger a binary 1 and not low enough to trigger a binary 0. Although if you remember the 5 voltage points of 1000Base-T the end of [Back to Bits](#back-to-bits) which makes the +-0V value trigger a binary digit, the IPG of 1Gbps or higher ethernet is made up of IDLE symbols like 0x90 NOP in x86 based assembly.

To summarize interpacket gap, think of it like the slight pause between each verbally spoken word. You need to enunciate clearly or your words will **collide**, and I'm using collide on purpose here to segue into the next section.

### Establishing an Ethernet Link: Speed, Duplex, and Auto-MDIX

There's a couple thing we need to cover before talking about how Media Access Control forms an Ethernet connection between two NICs. I'll be starting with the more logical portion then progress towards the more physical side of the connection like how I covered the portions of the frame in the last two sections.

Speed and Duplex for the most part go hand in hand. Most of what speed is I covered in [Back to Bits](#back-to-bits), think 10BASE-T, 100BASE-T and TX, 1000BASE-T, etc, so in the context of Speed and Duplex together we're talking about the rate of transmitting bits at the given method. Duplex on the other hand deals with whether or not the NICs can send and receive at the same time.

There's half-duplex and full-duplex. Both methods still allow each NIC on either side of the CAT5 cable to send or receive, this isn't token ring where one cable is for sending and one is for receiving-- remember we're focusing on Ethernet here, but half-duplex means there is only one communication direction across the cable at each given time. The NICs basically take turns talking and listening. Full-duplex on the other hand allows both NICs to send *and* receive at the same exact time which gives bi-directional communication.

For an easy comparison, if both of us have walkie talkies I have to let go of the transmit button when I'm done speaking to allow you to speak (half duplex) but if we're using our cell phones we both can scream and hear each other's screams at the same time (full duplex).

The way this works in Ethernet with a copper CAT5e cable hinges on how each NIC is using the twisted pairs inside that cable. Using 100BASE-TX Fast Ethernet as the example standard we have a maximum of 2 pairs being used out of the total 4 pairs inside the CAT5e cable (specifically pins 1,2,3, and 6). Full-duplex will use one pair for each communication direction which means both NICs are getting 100Mbps of input and output bits being transceived simultaneously. Half-duplex only utilizes one pair, so if NIC A is sending and NIC B is receiving, there is still a transmission rate of 100Mbps but only for that one direction. Once NIC A has taken its turn to transmit it then allows NIC B to transmit and now we have 100Mbps in the *other* direction. It's a bit easier to visualize this so here's a diagram showing both types of connections transmitting two frames:

![duplex diagram](assets/images/duplex-diagram.png)

If you're reading between the lines you're going to see why speed is ambiguous without context. Speed in the context of Speed and Duplex refers to the bandwidth of the connection, so the maximum amount of data (bits in this case) that can theoretically be transmitted across the connection. This means we can expect a 100BASE-TX connection to have the capacity to transfer 100,000,000 bits in one second. When someone complains about slow speeds it's not a 100BASE-TX suddenly modulating voltage at a slower rate, making the raw transfer of bits somehow slower, they're actually feeling lower throughput.

Throughput is what data actually gets transmitted in a given amount of time. A number of things can influence throughput like latency due to distance or the time a router/switch/server/whatever takes to process the data and respond. Lowered throughput in the context of this section is going to come from the connection operating at half-duplex.

To beat a dead horse there is still the capacity to transmit 100,000,000 bits in a given second but both NICs are taking turns sending and receiving which essentially halves the throughput. Take all the concepts we covered so far like overhead, forced interpacket gap and whether or not there was even data to send and throw it all out just for one moment. If we look at it like NIC A sends a bit, then NIC B sends a bit, back and forth for one solid second using a bandwidth of 100Mbps, that means NIC A only had 50,000,000 bits of transmitted throughput while it operated at half-duplex.

And fun fact, while wired connections are pretty much always operating in full-duplex if configured correctly, 802.11 WiFi can only work at half-duplex.

That's why it's hilarious to see WiFi players be salty on ScrubQuotes when they try to defend their connection after fighting games have finally started adding WiFi indicators or WiFi only lobbies. Sorry, but your Shoryuken input is taking twice as long, if not longer due to all that 802.11 overhead *(See! there was a reason for that second table!)*, and that's why the netcode is freaking out with delay or rollback.

Now lets talk about wiring and terminating those pairs with this diagram I'm pulling from Imgur from an original source I can't actually figure out, so credit goes to RWH in 2007

![rj45-pinout](https://i.imgur.com/qNYY7Gw.png)

To refresh what's been stated briefly and erratically, 10Mbps and 100Mbps connections only use two pairs of wires which ends up being pins 1,2,3, and 6 while 1Gbps or higher uses all four pairs. T-568A and T-568B are the two standardized pinouts for which colored wire goes to which pin, the only real difference between the two is solid green and solid orange, pins 2 and 6, swap places. Even though you can cut the jacket off a cable and see the *ever so slight twist rate* between the different colored pairs, there is no significant performance difference between the two. T-568A just has better backwards compatibility with older USOC pinouts since it can support two pairs with one cable instead of T-568B supporting only one pair per cable.

So in the name of backwards compatibility in existing older infrastructure, the federal government still requires T-568A termination. T-568B on the other hand matches AT&T's old 258A color code so it's just what everyone else kept on using. It boils down to which flavor of Bell Labs did you rep in the day, public sector or private sector? So obviously I'll be using T-568B in all examples as it's the ~~superior~~ most commonly used pinout.

Depending on what kind of devices you are connecting you may use both pinouts though. When connecting "like" to "like" a crossover cable is required, and that just means one side of the cable is terminated with T-568A and the other terminated with T-568B. Take a look back at the pinouts and specifically which pins transmit (Tx) and receive (Rx) and the colors associated.

In T-568B the orange and white orange wires, which is one pair, are both used for transmitting and the green and white green wires, which is the second pair, are both used for receiving. If you terminate a cable with T-568B on both sides then the Tx and Rx pins have their wires ran **straight through** to the same Tx and Rx pins. So when you connect two "like" devices you need to **cross over** the paired wires so each pair of pins gets a Tx and Rx wire.

When I say two "like" devices I'm using the way I learned and remembered the shorthand of the real terms. While it's not actually the full device that's alike, it's the interface. Layer 3 nodes on the network like PCs, servers, and routers all use Medium-Dependent Interfaces, MDI, while layer 2 switches use Medium-Dependent Interface Crossover, MDI-X. The functional difference between MDI and MDI-X is the crossover is done internally on a device using MDI-X, meaning the cross over in Tx and Rx pins is done on the NIC of a switch instead of changing the wire termination by making one side T-568A and the other T-568B

To summarize this:

- Router to Router is "Like to Like", requiring a crossover cable
- PC to PC is "Like to Like", requiring a crossover cable
- Router to PC is "Like to Like", requiring a crossover cable
- Switch to Switch is "Like to Like", requiring a crossover cable
- Router to Switch is **not** "Like to Like", requiring a straight through
- PC to Switch is **not** "Like to Like", requiring a straight through

If someone is less familiar with straight through and crossover cables and starts wondering why pretty much all "premade ethernet cables" don't list  if they are straight through or crossover, or the ones that do may just say straight through or T-568B, that's because we got smart and said hey we'll just automate that process. Thank HP for coming up with auto-MDIX which lets two NICs work some magic and *negotiate* if any pins need to be crossed over and which NIC internally crosses over which PIN.

Aaaaand we're back to speed and duplex because now let's talk about auto-negotiation. Before IEEE 802.3u Fast Ethernet was standardized,100BASE-TX on copper twisted pair cables, speed and duplex needed to be manually hardcoded. That means the first time you physically connect two devices or if you wiped the config on one device you had to pop into either the CLI or GUI of each device and set it for 10 or 100 Mbps along with half or full-duplex. If it's a router or switch you pulled fresh out of the box you're going to you need to whip out that light blue rollover cable and plug into the console port and hope the default baud rate is 9600 because you threw out the manual with the box so you can't look and see its 115200 baud.

Then, because the ISP circuit is a T1 and the router which you can't get into is the only device with a T1 interface, you can't exactly just plug a PC in directly and look up the default baud of that model online. Of course this is purely a hypothetical scenario and neither I nor anyone else reading this has lived through this hell or the equivalent hell of wiping your bootloader when trying out a new distro only to realize the CD (or dear god one of a dozen floppies) is bad, forcing you to see what friend with a computer is still awake and is okay with you sprinting to their house so you can use it to download and burn a new install disk. Totally hypothetical.

Thankfully those days are, for the most part, behind us. Network interfaces by default are configured with auto-negotiation turned on as well as auto-MDIX. You still need to configure whatever IP addresses to get CLI and GUI access but the Ethernet connection will form automagically. Before we talk about auto-MDIX we need to talk about speed and duplex auto-negotiation since it came first and auto-MDIX almost piggy backs off it's development.

When a NIC gets physically connected to another NIC, assuming MDI/MDI-X is fine, it will first start transmitting a 1 bit roughly every 16ms which is called a *normal* link pulse, or NLP. This is part of the Link Integrity Test to make sure it's properly connected to another NIC. After the two NICs confirm basic connectivity by receiving each other's NLPs they will either start sending frames if they are already hard coded for speed and duplex.

When speed and duplex is not hard coded and instead the configuration of each newly connected interface is set to auto-negotiate they will then send a burst of NLPs in a specific coded pattern. This burst is called the Fast Link Pulse, or FLP, and instead of being just binary 1's and silence that pattern is 16 bits of 1's and 0's sent in a ~2ms time frame and then the ~16ms gap of silence.

The pattern of 1's and 0's used in the FLP signify what capabilities that NIC has and is willing to use. The first 5 bits are the offer for which standard to select with one of them being 802.3 Ethernet. After that is the technology field which when 802.3 is chosen will offer which speed and duplex, so one bit is whether or not the NIC supports 10BASE-T half-duplex, another bit for 10BASE-T full-duplex, and this continues for 100BASE-TX half, 100BASE-TX full, etc. Each NIC will state it's full capability which means a Fast Ethernet interface will still "offer" 10BASE-T half duplex.

The NICs will then see where the overlapping support is and choose the best speed and duplex by weighing the possibilities by highest speed and preferably full-duplex. The other bits in the FLP deal with acknowledging the other's FLP, if there is suddenly a link failure, or if there is going to be another "page" of FLP because the NIC has more capabilities than can fit on that single 16 bit FLP.

This may sound out of left field but I as I write this, and anything technical for that matter, I fact check pretty much everything as I write it. Sure I know the material and can give a higher level run down as well as have some numbers completely burned into my brain but I'm human and may misquote a number or only remember 75% of the concept. That's not good enough for me when I'm trying to teach someone, I'd be doing a disservice to them (or you the reader) if I'm not absolutely correct while covering every small detail. The reason I'm bringing this up is I had originally written a few paragraphs talking about the extensions in auto-negotiation that were added in 1000BASE-T and higher.

The other reason is when explaining the potential issues with 1000BASE-T auto-negotiation versus hardcoding, in regards to the fact it still uses two pairs to negotiate speed and duplex and the new Master/Slave determination, I tried to find the hard proof of why you shouldn't hard code or mix and match hard coded and auto negotiation when connecting an interface that's capable of 10/100/1000Mbps to one that is only capable of 10/100Mbps.

Bouncing through the 5600 page 802.3-2018 document and reading Clause 28, Clause 40, and anything else that may be relevant to auto-negotiation on 1000BASE-T, I couldn't find the "if you connect these two and configure it this way it'll break" like I'd normally find in an IETF RFC.  So I hit google and poured over forums and publicly accessible lecture slides and pretty much all of them said it would just negotiate 100/Full.. but over the past several years I've seen these two types of interfaces fail negotiation to 100/Half as well as even hard coding both sides to 100/Full makes it operate in 100/Half.

Then I found a post on a now defunct forum dedicated to building your home lab for studying Cisco certifications, only accessible through archive.org's wayback machine, that completely vindicated my experiences and disproved everyone that said they tested with two Catalyst switches and they worked at 100Mbps fine.

In the interest of length/beating a dead horse here, please check out that [forum post snapshot](https://web.archive.org/web/20090226221950/http://www.ciscohomelab.com/learning/do-not-force-gigabit-ethernet-auto-negotiation-to-1000full/) because it really hits home with what I like to call "Pulling a Cisco" where in the interests of being first to market with a fancy new piece of tech,in this case 1000BASE-T, the vendor will just wing it when it comes to the inner workings that most likely will end up being *not at all* how it will be detailed in the standard or spec once it's been finalized by IEEE, IETF, ITU, or whoever else.



Next, and thankfully simpler and therefore shorter, is auto-MDIX

When you first connect the two NICs they will independently use an 11-bit linear-feedback shift register to create a pseudo-random number, the math behind it is like the polynomial algebra in CRC32. Once the NIC generates that number it uses it as the seed for choosing to be in MDI or MDI-X mode. With the first choice made of MDI or MDI-X it waits and listens 55ms to see if it hears a link or data pulse sent by the other NIC, if it does hear it that means it's good and will keep that original MDI or MDI-X choice. If it does not hear it then it will increment it's LFSR and make a choice on being MDI or MDI-X all over again based on pseudo-random number+1.

While the NIC is listening and switching from MDI to MDI-X or vice versa it's also sending link pulses so that the other NIC can do the same. This process runs until both NICs confirm the Tx and Rx pins which takes 1-2 cycles at 50ms each for connections below 1000BASE-T. On 1000BASE-T and higher connections using all 4 pairs will take longer, since there's two more pairs to work out between each NIC on the connection, but it's not **long**. In most cases 1000BASE-T or 10GBASE-T, which needs a Category 6 cable at minimum but ideally CAT6a so it reaches 100 meters, will go through the auto-MDIX process in under 500ms but sometimes take 1.4 seconds. *Sometimes being a 1 in 5,000,000,000,000,000,000,000 chance.*

After auto-MDIX does it's thing then the auto-negotiation process for speed and duplex takes place that was covered directly above. Additionally if you read that forum post snapshot about 1000BASE-T negotiation issues, **WHICH I HOPE YOU DID**, not only is auto-negotiation built into the *finalized* 1000BASE-T standard but so is auto-MDIX.

With all that being said, what happens when speed and duplex mismatch and why does it impact speeds? The devices on the network, or rather their NICs connected on the same medium, will try to transmit at the same time which makes their frames collide. Which brings us into the last section of Media Access Control.

Hooray, only one more section for MAC!

### Collisions and How To Stop Them with CSMA/CD and CSMA/CA

Carrier-sense Multiple Access, CSMA, is a protocol I've explained and alluded to *without actually naming it yet.* Whenever a node on a shared medium, 2 or more NICs on the same cable, are listening and reacting to the signals generated by the other node they are using Carrier-sense. The interpacket gap, listening for a basic connection with SLP, any synchronization done by modulating the wire voltage-- all of that is pulled together as a function of Carrier-sense.

The whole goal of Carrier-sense is to eliminate collisions which is when two nodes transmit at the same time. To explain collisions I'm going to bring up the forgotten red headed stepchild of network devices that still sometimes pops up in use today, the layer 1 hub. While routers operate on OSI layer 3, switches operate on OSI layer 2, hubs operate on OSI layer 1. There is absolutely no underlying logic or checks and balances like encoding or Carrier-sense or anything like that when a hub is operating, it's simply a signal repeater. If a hub has 4 ports and a device is connected to each of them, anytime one device transmits any kind of signal, even a link pulse, the hub replicates that on the three other ports 1:1 exactly how it received it.

When device 1 is trying to communicate to device 2 and ,simultaneously, device 3 is trying to communicate to device 4 they will each use their respective NIC to change the voltage and generate a binary signal which the hub receives at the same time and then try and propagate *both* signals to devices 2 and 4. This makes the bits "collide" and effectively destroy the frames being sent to those other devices.

To quote my instructor back in college, "hubs is stupid".

With just basic Carrier-sense Multiple Access there is no "what do we do now" after a collision happens. CSMA/Collision Detection then modified added more steps to the initial "am I ready to transmit a frame? If yes, is no one else transmitting a frame?" that is done by basic CSMA. If a collision does occur CSMA/CD pivots to a separate set of steps. Both are seen in the following flow chart originally from Wikipedia but I changed the line color to be more legible:

![CSMACD-Algorithm-legible](assets/images/CSMACD-Algorithm-legible.svg)

<cite>sourced from https://upload.wikimedia.org/wikipedia/commons/3/37/CSMACD-Algorithm.svg with modified colors</cite>

There's two things I'd like to add to this chart, the additional 12 bytes of waiting that is added to the "Is some other station transmitting?" check when using 802.3 Ethernet, and what happens during the wait period. Once the logic flow moves to wait <- rand(16) what is actually happening is the NIC that detected a collision is blasting a jamming signal across the wire where the collision occurred.

This jamming signal is a 16 bit pattern, hence the (16), that the NIC basically yells at anyone else listening to "stop, hold up, wait a minute, something ain't right". If the collision occurred before the minimum frame size of 64 bytes then it will keep repeating the jam signal until that is met. The NIC then increments the attempt counter and tried again until it is either successful or collisions kept occurring and it reached the max attempt limit of 15 tries.

The collision that happened here is called a *local* collision because the transmitting NIC detected it locally. There is also *remote* collisions when the NIC receiving the frame does it's cyclic redundancy check and sees what it calculated does not line up with the value in the FCS field, so the receiving NIC discards it because the frame is corrupted.

The key difference with a remote collision is the NIC that sent the frame *does not know the remote collision occurred* and therefore will not transmit that exact frame again. The data encapsulated inside will only be sent again if a higher layer protocol is used that would be aware of that and send the data again but as a completely different frame.

In today's world collisions on wired ethernet are pretty much eliminated. There's not a huge price difference between a switch and a hub, if you can even find a newly made hub that works on 100Mbps or 1Gbps, and half-duplex is seen as an issue and not a design choice. CSMA/CD is still actively running on wired ethernet connections but silently in the background waiting for someone to dig through their stock of devices made in the early 90's and pull a 10BASE-T hub out because they want one extra port without spending a dime on an additional switch. It still happens and CSMA/CD will try it's best but I've only seen a customer pull a hub out maybe half a dozen times in the past several years.

With 802.11 WiFi collision handling is done differently. Instead of CSMA/Collision Detection it uses CSMA/Collision Avoidance. The first difference between the two is how the collision domains get broken up. In a wired network the entire layer 1 run between interfaces is the collision domain, so if there is a hub that is metaphorically fusing the cables of 4 different devices together that is all one collision domain while a 4 devices plugged into a layer 2 switch would have 4 separate collision domains. Since the medium is now the open air (and soft objects) there isn't a way to physically separate the connection between each device. The world is one big unpowered hub. Instead CSMA/CA leverages the separate radio channels that 802.11 creates.

CSMA/CA will first listen to the channel and see if it is **receiving** anything on that channel. Due to signal strength and a 3-dimensional medium there is no way for the wireless node to see if that channel is in use by some other node closer to the destination.

For example we have nodes A--B--C in a line. Nodes A and C have the signal strength to go juuuuust up to node B but the signal dissipates before reaching the other, making those nodes *hidden* to each other. Node A can listen to channel 1, spitballing a number, and see it's free and start transmitting what it wants to say to node B who is already receiving node C on channel 1. This is the Carrier-sense part of it. Next is Collision Avoidance and if the node hears someone else talk on the channel it wants to use it will back off and wait a random amount of time and then listen again. If it's now free then it transmits whatever it needed to say because it avoided any potential collisions.

There are some extra options depending on the flavor of 802.11 as well as what hardware is used that can improve this process. To get around the hidden node problem you can use Request To Send/ Clear To Send where node A who thinks the channel 1 is free asks node B if it's *really* free and okay to transmit on it. Node A will then keep spiraling out waiting and listening periods until it hears node B say it's allowed to transmit. When node A never hears an answer and loops through backoff timers it's because it thinks it's request to send collided with a different node's transmission and prevented it from ever reaching node B.

For better or worse I'm not going to bring up new things about 802.11 or speak further on it here, this is already a sizeable info dump and we aren't even in Layer 3 yet. In the future I may write a post about the other methods and protocols involved like MIMO or 2.4GHz vs. 5GHz but this isn't the place for that.

## Logical Link Control

We're finally into the last section of new information in regards to OSI Layers 1&2 / TCP/IP Layer 1 and it's going to be a brief one. While there is a lot that can happen at the Logical Link Control sublayer it's pretty straight forward for Ethernet. LLC's depth really comes form every 802 standard that *isn't* Ethernet like Point to Point Protocol over ATM or SONET.

What was said before about Ethertype and Length is what will be covered and expanded upon here along with a quick synopsis on multiplexing and how flow control and error handling can be done by LLC.

### 802.2 LLC and SNAP

Before talking about that extra 8 bytes of header thats in the IEEE 802.3 frame, specifically not Ethernet II, we need to talk about the EtherType/Length field found in both of those frame types. This field is used to determine what the frame's payload is supposed to be **or** it's length.

This determination comes from the value of the field, if the value is hex 0x0600, decimal 1536, or higher then it is seen as an Ethernet II frame and the value translates to an EtherType which just describes what protocol is encapsulated in the frame. Some EtherTypes being 0x0800 as IPv4, 0x86DD as IPv6, 0x8847 and 0x8848 as MPLS unicast or multicast respectively, and 0x0806 as ARP.

If the value is 1500 or lower then it's seen as an 802.3 frame where the value is the length, 1500 being the max MTU of an 802.3 frame, and then it must be followed by a 3 byte LLC header, also called an LSAP, then a 5 byte SNAP header. In order, those 8 bytes are:

- DSAP - 1 byte - Destination Service Access Point, the upper-layer related station/node/host/etc addresses
- SSAP - 1 byte - Source Service Access Point, the upper-layer related *entity* source
- Control - 1 Byte (sometimes 2 bytes) - Used for connection and flow control for the upper layers but done at this lower layer
- OUI - 3 Bytes - Organizational Unique Identifier - Either an EtherType value like in Ethernet II frames or an organization like Oracle for Oracle's in house use of whichever upper layer protocol they registered it for, think how TCP/UDP port 666 is registered for Doom
- Protocol ID - 2 Bytes - EtherType value again

So there's a bit of redundancy, especially with the use of EtherTypes in the OUI and Protocol ID field which in of itself overlaps with EtherType in Ethernet II.

For a quick explanation, the combination of DSAP, SSAP, and Control form the LSAP which was implemented before the SNAP header. In these early days, the DSAP contained the higher level protocol's destination address, so the same idea of listing a destination IP address in a layer 3 header but it's in the layer 2 header.

Then the SSAP byte has it's first bit signify if the frame's encapsulated payload contains either command being issued or the response to an issued command. The last 7 bits in the SSAP then denote what the actual service or entity or whatever you want to call it of that encapsulated data, for instance 0x42, decimal 66, means the protocol being used is spanning tree while 0xF0 is IBM NetBIOS.

Well 7 bits can only make a range of numbers that total to 128 and if we're bringing the idea of a layer 3 packet down to layer 2, there is a waaaaay higher amount of protocols that can be transported with IP in the traditional sense than just 128.

During LLC without SNAP's development, they actually thought that 128 protocols would cover what every vendor would ever need.... yeah.... that held up *fiiine*. SNAP gets introduced to solve that limitation by going (and this is obviously me making up what was said) "Okay, since a single vendor like Oracle, Novell, or IBM could take up 128 protocols alone we'll add a 3 byte OUI to say this is Whoever's protocol and then offer a slew protocols to choose from in the last two bytes, that'll future proof it!"

And to be fair it did.... Except we moved towards TCP/IP which, due to the requirement of high interoperability to carry a wide range of higher level data over a wide range of lower level infrastructure. While IP over Ethernet totally works on 802.3 frames it absolutely requires the SNAP header so ARP can be supported which is a fundamental aspect of TCP/IP that we'll get into with the Layer 3 section. Since the LSAP and SNAP bytes eat into the payload as 802.3 frames can't exceed a certain size we're now dealing with required bloat and limitations at the layer 2 level when we just want to make it as streamlined as possible in an entirely TCP/IP environment using just Ethernet.

So Ethernet II regresses on the wide array of protocols supported by LSAP and SNAP, since we want just IP over Ethernet as it's the clear winner in having devices communicate, and then take the idea of a protocol ID and make that the only field we **add** onto the header with. Plus ditching support for dead protocols means we can build towards getting larger frames.

That's an extremely boiled down, cliff notes, not-entirely-correct explanation of what happened with 802.3 frames, why Ethernet II frames came to be, and way they both work. Unless you're using legit ARPANET, I don't think explaining it any further than that would be beneficial here.

### Muxing on Shared Media

Multiplexing is such a cool concept to me, it's basically taking multiple distinct signals and finding a way to combine them just enough to go across a single piece of physical media but then be able to rip those signals back apart into their distinct forms.

If you're coming into this concept blind with no knowledge whatsoever I think time-division multiplexing, TDM, is the easiest to wrap your head around. Time-division means we're, well, dividing how the signals travel based on time. The nitty gritty of how it works can vary based on what the standard or application is, but they all work on the concept of sending the message or part of a message from conversation 1 first, then conversation 2's message/partial message, then conversation 3's, etc repeated for however many distinct signals or conversations there are, before looping right back around.

The signals are kept separate by making sure each one has exactly the same amount of time to transmit and it's done in a circular pattern. You can think of it like dealing cards to your friends at the poker table but you hand each person their card in 1 second intervals as you go around clockwise or counter clockwise because it's the easy way to make sure everyone gets the right and same amount of cards.

There's other forms of muxing too like frequency-division multiplexing where you have the oscillating wave sent across the wire which is the carrier signal. The carrier signal then gets modulated so on that waveform there are little tiny waveforms that make up the data being sent. By clipping off the carrier signal on the other side with some form of band filter to leave just the tiny waveforms that rippled along the carrier wave you get the data signal back.

FDM can carry multiple signals too if the distinct signals get modulated to noticeably different frequencies on the carrier wave, then you filter off the carrier wave and are left with with some tiny waveform ripples at this frequency and can see there are other tiny waveform ripples at a completely different frequency and when the filter gets narrower in scope for one specific frequency it just then looks like any other data signal. Repeat for however many different signals on however many frequencies and you get all the different conversations.

In current practice, you'll see muxing happen with every form of wireless communication, fiber optics, and old telco infrastructure like ISDN using T-carrier circuits (especially when a T3 is splitting/muxed into 28 T1s).

To quote Marge Simpson, "I just think they're neat."

### Flow Control and Error Handling

This is an easy couple sentence section.

LLC pretty much doesn't touch Flow Control or Error Handling, save for a handful of protocols:

- Microsoft NetBIOS
- FDDI, which essentially died after FastEthernet came out
- X.25, from all the way back in the 1970s.

Apart from those minor exceptions, it's safe to assume that anything regarding Flow Control or Error Handling is performed at a layer higher than OSI's Layer 2 or TCP/IP's Layer 1.

## TCP/IP Link-Layer & OSI Layers 1+2 in Summary

Let's get this all wrapped up and move on.

In short:

OSI's Layer 1, the physical layer, deals with just the physical cabling and movement of data. No decision making is done there, unless you consider consider encoding & decoding and modulating & demodulating signals as a decision. I do not as they have no bearing on path selection.

OSI's Layer 2, the data-link layer, deals with the lowest level of decision making as seen by an ethernet switch referencing it's CAM table. Layer 2 decision making is restricted to the local area network which extends up until the next Layer 3 network interface on a router or PC.

TCP/IP's Layer 1, the link layer, combines those two OSI model layers because Ethernet operates in both and the OSI model wasn't designed specifically for how a hierarchy built on the Internet Protocol stack running over Ethernet is presented. Conceptually the TCP/IP model, aka the DoD model, works better for modern networks that pretty much all use IP over Ethernet.

The depth and complexity that can happen here is seemingly endless, but for the most part everything has moved towards those two common standards. Knowing the intricacies of how the different subsets of Ethernet like 100BASE-TX vs 10GBASE-T is useful but not crucial. It's more important to know what works with what as well as any limitations, such as speed+duplex caveats in 1000BASE-T when connected to 100BASE-TX, max cable lengths, whether or not you need CAT6 instead of CAT5e, and how to approach a possible issue.

For the most part you just need a solid understanding of the concepts and then go deeper into a specific subset of concepts if the need arises/your day to day work requires it. That solid understanding will benefit you elsewhere even when you're not in charge of a LAN because knowing overhead and MTU sizes will help a security engineer who deals with tunnels all day, an IPsec tunnel would introduce even more overhead so in some scenarios they may need to take MTU into account when there are throughput or stability issues due to fragmentation.

I'll end up repeating this, but every layer builds off each other. An issue in one will cause an issue in another if not outright downtime.

### Common Physical/Link Issues and Things to Keep in Mind

- First level of logic/decision making to allow two devices to communicate, building ontop of layer 1 which is only concerned about the literal movement of signal
- MAC addresses are burned into each individual network interfaces on a device by the manufacturer (identified by the first 3 octets in the address)
- Decision making is based on source and destination MAC addresses
- Source and destination MAC addresses will remain unchanged while the frame is in transit across the layer 2 domain, only being modified when the frame is received by a layer 3 router-- but not by any of the layer 2 switches (more on that in the next section)
- Most common issues derived from misconfiguration or miswiring
- Security wise--  Port Security, MACSEC, BPDU Guard, DHCP Snooping, and other implementations for layer 2 switches are more about network stability and preventing outages than preventing deliberate network breaches. Not saying they aren't used for actual security but they shine in stopping PEBKAC.

# Wrapping Things Up for Now

That was probably more than anyone wanted to know about the phyiscal and link layers, coming in at... sheesh... over 23,000 words.

## A Change of Plans

I had originally planned to have this first post be layers 1, 2, *and* 3-- even writing most of the content for  layer 3 prior to this "wrap up" section. My idea was to use the gap between layer 3 and 4 as the perfect stopping point because it's when the OSI model starts to transition out of purely network infrastructure and moves towards a focus on the methods of transporting client-side specific data *through* the network.

The issue I'm having is length and time. I finished writing the above summary section for TCP/IP in October after re-starting this entire blog idea around February.. of 2021. Now that February of 2022 has drawn to a close, I've made the decision to split this entire write up a second time so that part 1 is Layers 1 and 2, part 2 is Layer 3, and then part 3 would be Layers 4-7.

Outside of "hey it's been a year" being a thought that is occurring more frequently, Layer 3 is turning out to be extremely long. Almost too ridiculously long. That one line opening to this wrap up section is something I pointed out for myself because, in comparison, I've written over 50,000 words for Layer 3 so far and I believe it will take *at minimum* another 5,000 words to finish it which would put this first blog post into novel-sized territory.

Whenever I write technical documentation, be it for my team or for personal use, I always worry about readability. All the documentation in the world won't help if it looks too overwhelming or if it wasn't written cohesively-- documentation needs to be an *intelligible* source of knowledge. My main issue is that I  generally re-write each sentence or paragraph two or three times to make sure I'm wording it correctly. I'm caught up in presenting the technical knowledge as thorough as possible while wrapping around a layer of context written in almost layman's terms.

It's a balancing act. On one side is the depth and breadth of the subject. This gets counterbalanced by making it more approachable in a way that would make Plato nostalgic for his Allegories and Metaphors.

## Moving Forward

Hopefully that clears up any questions about my decisios to split this series into three parts-- and gives me the confidence to release this first post without Layer 3.

At the time of writing *this* closing section, there's a few more technical/new information sections to write in regards to Layer 3 before moving on to the proof reading and editing process.

I have two routing protocols left to cover, then three shorter sections that are going to be written more like primers on their respective topics, and then there's the closing summaries where I like to boil all the material down into a condensed list of what key takeaways. In addition to those sections I'd like to write out a demonstration that pulls together everything that happens in the first three layers of the OSI model to really solidfy the material and then cap it all off with a list of the best learning material that is held in almost reverence among my colleagues and I.

Thanks for taking the time to read this.
