# Cellular



## SIM

The [SIM](https://en.wikipedia.org/wiki/Universal_Subscriber_Identity_Module) (Subscriber Identity Module) card:

[UICC](https://en.wikipedia.org/wiki/Universal_integrated_circuit_card) (Universal Integrated Circuit Card): Refers to the integrated circuit
that composes the SIM.

- [IMSI](https://en.wikipedia.org/wiki/International_mobile_subscriber_identity) (International Mobile Subscriber Identity): A 64-bit (8 bytes)
  number that identifies the client. It has three parts:
  - [MCC](https://en.wikipedia.org/wiki/Mobile_country_code) (Mobile Country Code): The 3 first digits of IMSI that indicates
    the country.
  - [MNC](https://en.wikipedia.org/wiki/Mobile_network_code#International_operators) (Mobile Network Code): The next 2 or 3 digits that identifies the
    network operator (aka ISP for mobiles), like AT&T or Vodafone.
  - [MSIN](https://en.wikipedia.org/wiki/Mobile_subscription_identification_number) (Mobile Subscription Identification Number): The rest of the
    digits that identify the client.
    
  An example of IMSI is 310170845466094:
  - 310 is the MCC (country UnitedStates)
  - 170 is the MNC (network provider Sprint) 
  - The remain 845466094 identifies the client

- Ki key: Is the key used by the SIM to authenticate the client. For security it
  must never leave the SIM.

    
## Mobile Device

The device used to connect to the cellular network. It has several names like
User Equipment (UE) or Mobile Station (MS).

- [IMEI](https://en.wikipedia.org/wiki/International_Mobile_Equipment_Identity) (International Mobile Equipment Identifier): It is an universal
identifier for each device in the cellular network




## Resources

- [An Introduction to Cellular Security](https://opensecuritytraining.info/IntroCellSec.html) by Joshua Franklin


- https://z4ziggy.wordpress.com/2015/05/17/sniffing-gsm-traffic-with-hackrf/
- https://github.com/mapennell/hackrf-gsm
- https://github.com/ptrkrysik/gr-gsm
- https://github.com/Oros42/IMSI-catcher
