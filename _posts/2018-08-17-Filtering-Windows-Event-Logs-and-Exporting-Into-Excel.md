---
title: 'Filtering Windows Event Logs and Exporting Into Excel'
categories: [Sysadmin]
---

Microsoft seem to love making working with their own tools difficult. 

Today I had a task to report on external RDS access for a customer. They use an RDS server for both internal and external access, so my task was made more difficult due to needing to filter out a lot of irrelevant events. Once the events were filtered correctly, I then had to struggle to get this into a format the client could understand (ie an Excel table).

## Collecting the relevant information

I started off browsing through the Network Policy and Access Services log on the gateway server and pretty quickly found what I wanted: Event 6272, Microsoft Windows security auditing

![Event 6272 Example Details]({{ '/assets/images/2018-08-17-event-6272.png' | absolute_url }})

I was then able to browse to the Security event log and filter on event 6272 to give a list of entries for each login event.

## Exporting event logs into Excel

Now I had a nice list of events that I can browse through. Great! But how can I get this information into Excel? Again, this is easier said than done. At first I tried exporting to CSV, but Microsoft in their wisdom decided that all the data should be once column.

Exporting to XML and then opening up in Excel came close, but each item of the event data was presented on a separate row. Someone with magical Excel-foo might be able to reconcile that, but it was beyond me. 

The trouble is the XML generated is of the format `<Data Name='Foo'>Bar</Data>`

```xml
<Event xmlns='http://schemas.microsoft.com/win/2004/08/events/event'>
<System>
  <Provider Name='Microsoft-Windows-Security-Auditing' Guid='{54849625-5478-4994-A5BA-3E3B0328C30D}' /> 
  <EventID>6272</EventID> 
  <Version>1</Version> 
  <Level>0</Level> 
  <Task>12552</Task> 
  <Opcode>0</Opcode> 
  <Keywords>0x8020000000000000</Keywords> 
  <TimeCreated SystemTime='2018-08-16T23:27:53.831121700Z' /> 
  <EventRecordID>982021970</EventRecordID> 
  <Correlation /> 
  <Execution ProcessID='572' ThreadID='16364' /> 
  <Channel>Security</Channel> 
  <Computer>[redacted]</Computer> 
  <Security /> 
</System>
<EventData>
  <Data Name='SubjectUserSid'>[redacted]</Data> 
  <Data Name='SubjectUserName'>[Users login name]</Data> 
  <Data Name='SubjectDomainName'>[Users login domain]</Data> 
  <Data Name='FullyQualifiedSubjectUserName'>[Fully qualified login]</Data> 
  <Data Name='SubjectMachineSID'>S-1-0-0</Data> 
  <Data Name='SubjectMachineName'>[the name of the machine we want]</Data> 
  <Data Name='FullyQualifiedSubjectMachineName'>-</Data> 
  <Data Name='MachineInventory'>-</Data> 
  <Data Name='CalledStationID'>UserAuthType:PW</Data> 
  <Data Name='CallingStationID'>-</Data> 
  <Data Name='NASIPv4Address'>-</Data> 
  <Data Name='NASIPv6Address'>-</Data> 
  <Data Name='NASIdentifier'>-</Data> 
  <Data Name='NASPortType'>Virtual</Data> 
  <Data Name='NASPort'>-</Data> 
  <Data Name='ClientName'>-</Data> 
  <Data Name='ClientIPAddress'>-</Data> 
  <Data Name='ProxyPolicyName'>TS GATEWAY AUTHORIZATION POLICY</Data> 
  <Data Name='NetworkPolicyName'>General Connection Authorization Policy</Data> 
  <Data Name='AuthenticationProvider'>Windows</Data> 
  <Data Name='AuthenticationServer'>[redacted]</Data> 
  <Data Name='AuthenticationType'>Unauthenticated</Data> 
  <Data Name='EAPType'>-</Data> 
  <Data Name='AccountSessionIdentifier'>-</Data> 
  <Data Name='QuarantineState'>Full Access</Data> 
  <Data Name='QuarantineSessionIdentifier'>-</Data> 
  <Data Name='LoggingResult'>Accounting information was written to the local log file.</Data> 
</EventData>
</Event>
  ```
  
So Excel can't reconcile this to separate columns. 

What isn't beyond me is Regex. So I opened up the XML in my trusty Sublime Text editor and made the following find/replace:

Find: `<Data Name='(.*?)'>(.*?)</Data>` <br/>
Replace: `<$1>$2</$1>`

For those of you who don't understand regex, this changes the `<Data>` tag to instead be named based off the value of `Name`.

 ```xml
 <Event xmlns='http://schemas.microsoft.com/win/2004/08/events/event'>
 <System>
   <Provider Name='Microsoft-Windows-Security-Auditing' Guid='{54849625-5478-4994-A5BA-3E3B0328C30D}' /> 
   <EventID>6272</EventID> 
   <Version>1</Version> 
   <Level>0</Level> 
   <Task>12552</Task> 
   <Opcode>0</Opcode> 
   <Keywords>0x8020000000000000</Keywords> 
   <TimeCreated SystemTime='2018-08-16T23:27:53.831121700Z' /> 
   <EventRecordID>982021970</EventRecordID> 
   <Correlation /> 
   <Execution ProcessID='572' ThreadID='16364' /> 
   <Channel>Security</Channel> 
   <Computer>[redacted]</Computer> 
   <Security /> 
 </System>
 <EventData>
   <SubjectUserSid>[redacted]</SubjectUserSid> 
   <SubjectUserName>[Users login name]</SubjectUserName> 
   <SubjectDomainName>[Users login domain]</SubjectDomainName> 
   <FullyQualifiedSubjectUserName>[Fully qualified login]</FullyQualifiedSubjectUserName> 
   <SubjectMachineSID>S-1-0-0</SubjectMachineSID> 
   <SubjectMachineName>[the name of the machine we want]</SubjectMachineName> 
   <FullyQualifiedSubjectMachineName>-</FullyQualifiedSubjectMachineName> 
   <MachineInventory>-</MachineInventory> 
   <CalledStationID>UserAuthType:PW</CalledStationID> 
   <CallingStationID>-</CallingStationID> 
   <NASIPv4Address>-</NASIPv4Address> 
   <NASIPv6Address>-</NASIPv6Address> 
   <NASIdentifier>-</NASIdentifier> 
   <NASPortType>Virtual</NASPortType> 
   <NASPort>-</NASPort> 
   <ClientName>-</ClientName> 
   <ClientIPAddress>-</ClientIPAddress> 
   <ProxyPolicyName>TS GATEWAY AUTHORIZATION POLICY</ProxyPolicyName> 
   <NetworkPolicyName>General Connection Authorization Policy</NetworkPolicyName> 
   <AuthenticationProvider>Windows</AuthenticationProvider> 
   <AuthenticationServer>[redacted]</AuthenticationServer> 
   <AuthenticationType>Unauthenticated</AuthenticationType> 
   <EAPType>-</EAPType> 
   <AccountSessionIdentifier>-</AccountSessionIdentifier> 
   <QuarantineState>Full Access</QuarantineState> 
   <QuarantineSessionIdentifier>-</QuarantineSessionIdentifier> 
   <LoggingResult>Accounting information was written to the local log file.</LoggingResult> 
 </EventData>
 </Event>
 ```
 
 That looks better! 
 
 Now you can go ahead and open this up in Excel and it will show all  the columns you're after.
 
 Note that this will only work if all the columns match (ie all the events need to be the same type/ID).