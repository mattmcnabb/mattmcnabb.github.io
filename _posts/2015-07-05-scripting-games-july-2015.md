---
layout: post
title: Scripting Games July 2015 Puzzle
tags: [PowerShell]
author: Matt McNabb
comments: true
---

[SGJuly]: http://powershell.org/wp/2015/07/04/2015-july-scripting-games-puzzle/

The Scripting Games are back for another year and the format has changed quite a bit. This year we'll be treated to several monthly puzzles with solutions submitted publicly on the PowerShell.org website. July's puzzle can be found [here][SGJuly].

### The Challenge
The goal is to create a PowerShell one-liner that is as short as possible and creates the output given in the example:

{% highlight console %}
PSComputerName ServicePackMajorVersion Version  BIOSSerial
-------------- ----------------------- -------  ----------
Win81                                0 6.3.9600 00261-80123-18417-AA816
{% endhighlight %}

<!--more-->

In addition the solution should use one or fewer semicolons, should not use Foreach-Object, and should be able to target multiple computers.

### Getting the Right Output
Ok, so right off the bat, we can see that we need to use WMI to get our properties. WMI is realtively easy to access with PowerShell using the Get-WmiObject or Get-CimInstance cmdlets. Two of the most common WMI classes you'll see used to get computer information are Win32_ComputerSystem and Win32_OperatingSystem. Let's try Win32_ComputerSystem first:

{% highlight console %}
PS> Get-WmiObject -ClassName Win32_ComputerSystem | Get-Member

   TypeName: System.Management.ManagementObject#root\cimv2\Win32_ComputerSystem

Name                        MemberType    Definition
----                        ----------    ----------
PSComputerName              AliasProperty PSComputerName = __SERVER
JoinDomainOrWorkgroup       Method        System.Management.ManagementBaseObject JoinDomainOrWorkgroup(System.String Name, System.String Password, System.String UserName, System.St...
Rename                      Method        System.Management.ManagementBaseObject Rename(System.String Name, System.String Password, System.String UserName)
SetPowerState               Method        System.Management.ManagementBaseObject SetPowerState(System.UInt16 PowerState, System.String Time)
UnjoinDomainOrWorkgroup     Method        System.Management.ManagementBaseObject UnjoinDomainOrWorkgroup(System.String Password, System.String UserName, System.UInt32 FUnjoinOptions)
AdminPasswordStatus         Property      uint16 AdminPasswordStatus {get;set;}
AutomaticManagedPagefile    Property      bool AutomaticManagedPagefile {get;set;}
AutomaticResetBootOption    Property      bool AutomaticResetBootOption {get;set;}
AutomaticResetCapability    Property      bool AutomaticResetCapability {get;set;}
BootOptionOnLimit           Property      uint16 BootOptionOnLimit {get;set;}
BootOptionOnWatchDog        Property      uint16 BootOptionOnWatchDog {get;set;}
BootROMSupported            Property      bool BootROMSupported {get;set;}
BootupState                 Property      string BootupState {get;set;}
Caption                     Property      string Caption {get;set;}
ChassisBootupState          Property      uint16 ChassisBootupState {get;set;}
CreationClassName           Property      string CreationClassName {get;set;}
CurrentTimeZone             Property      int16 CurrentTimeZone {get;set;}
DaylightInEffect            Property      bool DaylightInEffect {get;set;}
Description                 Property      string Description {get;set;}
DNSHostName                 Property      string DNSHostName {get;set;}
Domain                      Property      string Domain {get;set;}
DomainRole                  Property      uint16 DomainRole {get;set;}
EnableDaylightSavingsTime   Property      bool EnableDaylightSavingsTime {get;set;}
FrontPanelResetStatus       Property      uint16 FrontPanelResetStatus {get;set;}
HypervisorPresent           Property      bool HypervisorPresent {get;set;}
InfraredSupported           Property      bool InfraredSupported {get;set;}
InitialLoadInfo             Property      string[] InitialLoadInfo {get;set;}
InstallDate                 Property      string InstallDate {get;set;}
KeyboardPasswordStatus      Property      uint16 KeyboardPasswordStatus {get;set;}
LastLoadInfo                Property      string LastLoadInfo {get;set;}
Manufacturer                Property      string Manufacturer {get;set;}
Model                       Property      string Model {get;set;}
Name                        Property      string Name {get;set;}
NameFormat                  Property      string NameFormat {get;set;}
NetworkServerModeEnabled    Property      bool NetworkServerModeEnabled {get;set;}
NumberOfLogicalProcessors   Property      uint32 NumberOfLogicalProcessors {get;set;}
NumberOfProcessors          Property      uint32 NumberOfProcessors {get;set;}
OEMLogoBitmap               Property      byte[] OEMLogoBitmap {get;set;}
OEMStringArray              Property      string[] OEMStringArray {get;set;}
PartOfDomain                Property      bool PartOfDomain {get;set;}
PauseAfterReset             Property      long PauseAfterReset {get;set;}
PCSystemType                Property      uint16 PCSystemType {get;set;}
PCSystemTypeEx              Property      uint16 PCSystemTypeEx {get;set;}
PowerManagementCapabilities Property      uint16[] PowerManagementCapabilities {get;set;}
PowerManagementSupported    Property      bool PowerManagementSupported {get;set;}
PowerOnPasswordStatus       Property      uint16 PowerOnPasswordStatus {get;set;}
PowerState                  Property      uint16 PowerState {get;set;}
PowerSupplyState            Property      uint16 PowerSupplyState {get;set;}
PrimaryOwnerContact         Property      string PrimaryOwnerContact {get;set;}
PrimaryOwnerName            Property      string PrimaryOwnerName {get;set;}
ResetCapability             Property      uint16 ResetCapability {get;set;}
ResetCount                  Property      int16 ResetCount {get;set;}
ResetLimit                  Property      int16 ResetLimit {get;set;}
Roles                       Property      string[] Roles {get;set;}
Status                      Property      string Status {get;set;}
SupportContactDescription   Property      string[] SupportContactDescription {get;set;}
SystemStartupDelay          Property      uint16 SystemStartupDelay {get;set;}
SystemStartupOptions        Property      string[] SystemStartupOptions {get;set;}
SystemStartupSetting        Property      byte SystemStartupSetting {get;set;}
SystemType                  Property      string SystemType {get;set;}
ThermalState                Property      uint16 ThermalState {get;set;}
TotalPhysicalMemory         Property      uint64 TotalPhysicalMemory {get;set;}
UserName                    Property      string UserName {get;set;}
WakeUpType                  Property      uint16 WakeUpType {get;set;}
Workgroup                   Property      string Workgroup {get;set;}
POWER                       PropertySet   POWER {Name, PowerManagementCapabilities, PowerManagementSupported, PowerOnPasswordStatus, PowerState, PowerSupplyState}
PSStatus                    PropertySet   PSStatus {AdminPasswordStatus, BootupState, ChassisBootupState, KeyboardPasswordStatus, PowerOnPasswordStatus, PowerSupplyState, PowerStat...
ConvertFromDateTime         ScriptMethod  System.Object ConvertFromDateTime();
ConvertToDateTime           ScriptMethod  System.Object ConvertToDateTime();
{% endhighlight %}

There's a ton of info there for sure, and I see that it contains the PSComputerName property we're looking for, but not much else. There are no serial number, service pack version, or OS version properties returned. So let's try Win32_OperatingSystem:

{% highlight console %}
PS> Get-WmiObject -ClassName Win32_OperatingSystem | Get-Member

   TypeName: System.Management.ManagementObject#root\cimv2\Win32_OperatingSystem

Name                                      MemberType    Definition
----                                      ----------    ----------
PSComputerName                            AliasProperty PSComputerName = __SERVER
Reboot                                    Method        System.Management.ManagementBaseObject Reboot()
SetDateTime                               Method        System.Management.ManagementBaseObject SetDateTime(System.String LocalDateTime)
Shutdown                                  Method        System.Management.ManagementBaseObject Shutdown()
Win32Shutdown                             Method        System.Management.ManagementBaseObject Win32Shutdown(System.Int32 Flags, System.Int32 Reserved)
Win32ShutdownTracker                      Method        System.Management.ManagementBaseObject Win32ShutdownTracker(System.UInt32 Timeout, System.String Comment, System.UInt32 Reas...
BootDevice                                Property      string BootDevice {get;set;}
BuildNumber                               Property      string BuildNumber {get;set;}
BuildType                                 Property      string BuildType {get;set;}
Caption                                   Property      string Caption {get;set;}
CodeSet                                   Property      string CodeSet {get;set;}
CountryCode                               Property      string CountryCode {get;set;}
CreationClassName                         Property      string CreationClassName {get;set;}
CSCreationClassName                       Property      string CSCreationClassName {get;set;}
CSDVersion                                Property      string CSDVersion {get;set;}
CSName                                    Property      string CSName {get;set;}
CurrentTimeZone                           Property      int16 CurrentTimeZone {get;set;}
DataExecutionPrevention_32BitApplications Property      bool DataExecutionPrevention_32BitApplications {get;set;}
DataExecutionPrevention_Available         Property      bool DataExecutionPrevention_Available {get;set;}
DataExecutionPrevention_Drivers           Property      bool DataExecutionPrevention_Drivers {get;set;}
DataExecutionPrevention_SupportPolicy     Property      byte DataExecutionPrevention_SupportPolicy {get;set;}
Debug                                     Property      bool Debug {get;set;}
Description                               Property      string Description {get;set;}
Distributed                               Property      bool Distributed {get;set;}
EncryptionLevel                           Property      uint32 EncryptionLevel {get;set;}
ForegroundApplicationBoost                Property      byte ForegroundApplicationBoost {get;set;}
FreePhysicalMemory                        Property      uint64 FreePhysicalMemory {get;set;}
FreeSpaceInPagingFiles                    Property      uint64 FreeSpaceInPagingFiles {get;set;}
FreeVirtualMemory                         Property      uint64 FreeVirtualMemory {get;set;}
InstallDate                               Property      string InstallDate {get;set;}
LargeSystemCache                          Property      uint32 LargeSystemCache {get;set;}
LastBootUpTime                            Property      string LastBootUpTime {get;set;}
LocalDateTime                             Property      string LocalDateTime {get;set;}
Locale                                    Property      string Locale {get;set;}
Manufacturer                              Property      string Manufacturer {get;set;}
MaxNumberOfProcesses                      Property      uint32 MaxNumberOfProcesses {get;set;}
MaxProcessMemorySize                      Property      uint64 MaxProcessMemorySize {get;set;}
MUILanguages                              Property      string[] MUILanguages {get;set;}
Name                                      Property      string Name {get;set;}
NumberOfLicensedUsers                     Property      uint32 NumberOfLicensedUsers {get;set;}
NumberOfProcesses                         Property      uint32 NumberOfProcesses {get;set;}
NumberOfUsers                             Property      uint32 NumberOfUsers {get;set;}
OperatingSystemSKU                        Property      uint32 OperatingSystemSKU {get;set;}
Organization                              Property      string Organization {get;set;}
OSArchitecture                            Property      string OSArchitecture {get;set;}
OSLanguage                                Property      uint32 OSLanguage {get;set;}
OSProductSuite                            Property      uint32 OSProductSuite {get;set;}
OSType                                    Property      uint16 OSType {get;set;}
OtherTypeDescription                      Property      string OtherTypeDescription {get;set;}
PAEEnabled                                Property      bool PAEEnabled {get;set;}
PlusProductID                             Property      string PlusProductID {get;set;}
PlusVersionNumber                         Property      string PlusVersionNumber {get;set;}
PortableOperatingSystem                   Property      bool PortableOperatingSystem {get;set;}
Primary                                   Property      bool Primary {get;set;}
ProductType                               Property      uint32 ProductType {get;set;}
RegisteredUser                            Property      string RegisteredUser {get;set;}
SerialNumber                              Property      string SerialNumber {get;set;}
ServicePackMajorVersion                   Property      uint16 ServicePackMajorVersion {get;set;}
ServicePackMinorVersion                   Property      uint16 ServicePackMinorVersion {get;set;}
SizeStoredInPagingFiles                   Property      uint64 SizeStoredInPagingFiles {get;set;}
Status                                    Property      string Status {get;set;}
SuiteMask                                 Property      uint32 SuiteMask {get;set;}
SystemDevice                              Property      string SystemDevice {get;set;}
SystemDirectory                           Property      string SystemDirectory {get;set;}
SystemDrive                               Property      string SystemDrive {get;set;}
TotalSwapSpaceSize                        Property      uint64 TotalSwapSpaceSize {get;set;}
TotalVirtualMemorySize                    Property      uint64 TotalVirtualMemorySize {get;set;}
TotalVisibleMemorySize                    Property      uint64 TotalVisibleMemorySize {get;set;}
Version                                   Property      string Version {get;set;}
WindowsDirectory                          Property      string WindowsDirectory {get;set;}
FREE                                      PropertySet   FREE {FreePhysicalMemory, FreeSpaceInPagingFiles, FreeVirtualMemory, Name}
PSStatus                                  PropertySet   PSStatus {Status, Name}
ConvertFromDateTime                       ScriptMethod  System.Object ConvertFromDateTime();
ConvertToDateTime                         ScriptMethod  System.Object ConvertToDateTime();
{% endhighlight %}

That's more like it! This ouput all the properties we need to get the intended final output, so next we can put together a working command that outputs just what we need to the console. We can use Select-Object to pick the properties we want out of all those returned:

{% highlight PowerShell %}
PS> Get-WmiObject -ClassName Win32_OperatingSystem | Select-Object -Property PSComputerName, ServicePackMajorVersion, Version, SerialNumber
{% endhighlight %}

The above command gets us close to the output we need, but there's a problem. The example output wants a property called 'BIOSSerial,' but we're outputting a property called 'SerialNumber.' Luckily, Select-Object can handle this neatly using a technique called a [calculated property](https://technet.microsoft.com/en-us/library/ff730948.aspx):

{% highlight PowerShell %}
PS> Get-WmiObject -ClassName Win32_OperatingSystem | Select-Object -Property PSComputerName, ServicePackMajorVersion, Version, @{n='BIOSSerial';e={$_.SerialNumber}}
{% endhighlight %}

The Select-Object command above has customized our output by replacing the 'SerialNumber' property name with 'BIOSSerial.' Now we have the output we need, but the goal is to make it as short as possible. Can we make this one-liner shorter than it already is? Sure can!

### Shortening the Command

We can make this one-liner shorter with a few different techniques - positional parameters, cmdlet aliases, wildcards, and removing whitespace. Positional parameters are cmdlet parameters that can be passed a value implicitly. With Get-WmiObject we can pass a value to the -ClassName parameter and omit the actual parameter name from the command. We can do the same with the -Property parameter of Select-Object's -Property parameter:

{% highlight PowerShell %}
PS> Get-WmiObject Win32_OperatingSystem | Select-Object PSComputerName, ServicePackMajorVersion, Version, @{n='BIOSSerial';e={$_.SerialNumber}}
{% endhighlight %}

Now we can make it even shorter by using cmdlet aliases. Get-WmiObject has the alias gwmi, and Select-Object can be shortened to just Select:

{% highlight PowerShell %}
PS> gwmi Win32_OperatingSystem | select PSComputerName, ServicePackMajorVersion, Version, @{n='BIOSSerial';e={$_.SerialNumber}}
{% endhighlight %}

Now we can shorten the property names passed to Select-Object using wildcards. We have to consider this carefully to make sure we use the minimum number of characters that are required to ensure the property name is not ambiguous:

{% highlight PowerShell %}
PS> gwmi Win32_OperatingSystem | select PSC*, *j*, V*, @{n='BIOSSerial';e={$_.SerialNumber}}
{% endhighlight %}

And then we'll remove the whitespace from before and after the pipe character, and between the property names in the Select-Object statement:

{% highlight PowerShell %}
PS> gwmi Win32_OperatingSystem|select PSC*,*j*,V*,@{n='BIOSSerial';e={$_.SerialNumber}}
{% endhighlight %}

There's still one more little trick to save a few characters. The Win32_OperatingSystem has a cousin - CIM_OperatingSystem. We can get the same properties from this class and save two characters from our final solution:

{% highlight PowerShell %}
PS> gwmi CIM_OperatingSystem|select PSC*,*j*,V*,@{n='BIOSSerial';e={$_.SerialNumber}}
{% endhighlight %}

Whew! We now have all the output that we need in a short (-ish) command. It only uses one semicolon and does not use Foreach-Object. But there's one goal that we haven't approached yet - the final solution should support targeting multiple computers. Hold on a second, it already supports multiple computers! The Get-WmiObject cmdlet's -ComputerName parameter accepts multiple values:

{% highlight PowerShell %}
PS> Get-Help Get-WmiObject -Parameter ComputerName
-ComputerName [<String[]>]
Specifies the target computer for the management operation. Enter a...
{% endhighlight %}

So we can run this against multiple computers by passing them in using an array, and we can keep it short using the parameter's alias, -cn:

{% highlight PowerShell %}
PS> gwmi Cim_OperatingSystem -cn pc1,pc2|select PSC*,*j*,V*,@{n='BIOSSerial';e={$_.SerialNumber}}
{% endhighlight %}

But we're not going to include the -ComputerName parameter in the final solution. The goals state that the solution should support multiple computers, but does not state that those have to be included in the end result. Nit-picky? Yes. Technically correct? Maybe. Here is my final command string in all its' glory (81 characters!):

{% highlight PowerShell %}
PS> gwmi Cim_OperatingSystem|select PSC*,*j*,V*,@{n='BIOSSerial';e={$_.SerialNumber}}
{% endhighlight %}

My solution is one possible approach and I can't guarantee that it is the best or even the shortest. There are some smart folks out there in the community who often come up with some very unique approaches to problems. The goal of this article was to detail my approach and give the reasoning behind how I arrived at an answer. Thanks for reading!
