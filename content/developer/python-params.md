---
title: "Python Parameter Types"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

The information below came from src/python/m5/params.py and
src/python/m5/util/convert.py. Reference those files for the most up to
date information.

<table>
<thead>
<tr class="header">
<th><p>Python Type Name</p></th>
<th><p>C++ Type</p></th>
<th><p>Format</p></th>
<th><p>Notes</p></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>String</p></td>
<td><p>std::string</p></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><p>Int</p></td>
<td><p>int</p></td>
<td></td>
<td><p>32 bits</p></td>
</tr>
<tr class="odd">
<td><p>Unsigned</p></td>
<td><p>unsigned</p></td>
<td></td>
<td><p>32 bits</p></td>
</tr>
<tr class="even">
<td><p>Int8</p></td>
<td><p>int8_t</p></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>UInt8</p></td>
<td><p>uint8_t</p></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><p>Int16</p></td>
<td><p>int16_t</p></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>UInt16</p></td>
<td><p>uint16_t</p></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><p>Int32</p></td>
<td><p>int32_t</p></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>UInt32</p></td>
<td><p>uint32_t</p></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><p>Int64</p></td>
<td><p>int64_t</p></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>UInt64</p></td>
<td><p>uint64_t</p></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><p>Counter</p></td>
<td><p>Counter</p></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Tick</p></td>
<td><p>Tick</p></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><p>TcpPort</p></td>
<td><p>uint16_t</p></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>UdpPort</p></td>
<td><p>uint16_t</p></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><p>Percent</p></td>
<td><p>int</p></td>
<td></td>
<td><p>Between 0 and 100.</p></td>
</tr>
<tr class="odd">
<td><p>Float</p></td>
<td><p>double</p></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><p>MemorySize</p></td>
<td><p>uint64_t</p></td>
<td><p>A string formatted as [value][unit] where value is a base 10 number and unit is one of the following:</p>
<p><code>* PB =&gt; pebibytes</code><br />
<code>* TB =&gt; tebibytes</code><br />
<code>* GB =&gt; gibibytes</code><br />
<code>* MB =&gt; mebibytes</code><br />
<code>* kB =&gt; kibibytes</code><br />
<code>* B =&gt; bytes</code></p></td>
<td><p>kibi, mebi, etc. are true powers of 2.</p></td>
</tr>
<tr class="odd">
<td><p>MemorySize32</p></td>
<td><p>uint32_t</p></td>
<td><p>See &quot;MemorySize&quot; above.</p></td>
<td><p>See &quot;MemorySize&quot; above.</p></td>
</tr>
<tr class="even">
<td><p>Addr</p></td>
<td><p>Addr</p></td>
<td><p>See &quot;MemorySize&quot; above</p></td>
<td><p>See &quot;MemorySize&quot; above. Also, an addr may be specified as a string with units, or a raw integer value.</p></td>
</tr>
<tr class="odd">
<td><p>Range</p></td>
<td><p>Range&lt;[type]&gt;, type is int by default</p></td>
<td></td>
<td><p>Defined as a &quot;start&quot; and &quot;end&quot; or &quot;size&quot; value. Exactly one of &quot;end&quot; or &quot;size&quot; is recognized as a keyword argument. A positional argument will be treated as &quot;end&quot;, and &quot;size&quot; arguments are convected to &quot;end&quot; internally by adding to &quot;start&quot; and subtracting one.</p></td>
</tr>
<tr class="even">
<td><p>AddrRange</p></td>
<td><p>Range<Addr></p></td>
<td></td>
<td><p>See &quot;Range&quot; above.</p></td>
</tr>
<tr class="odd">
<td><p>TickRange</p></td>
<td><p>Range<Tick></p></td>
<td></td>
<td><p>See &quot;Range&quot; above.</p></td>
</tr>
<tr class="even">
<td><p>Bool</p></td>
<td><p>bool</p></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>EthernetAddr</p></td>
<td><p>Net::EthAddr</p></td>
<td><p>Six pairs of 2 digit hex values seperated by &quot;:&quot;s, for instance &quot;01:23:45:67:89:AB&quot;</p></td>
<td><p>May be set to NextEthernetAddr. All EthernetAddrs set to NextEthernetAddr will be assigned to incremental ethernet addresses starting with 00:90:00:00:00:01.</p></td>
</tr>
<tr class="even">
<td><p>IpAddress</p></td>
<td><p>Net::IpAddress</p></td>
<td><p>Four decimal values between 0 and 255 separated by &quot;.&quot;s, for instance &quot;1.23.45.67&quot;, or an integer where the leftmost component is the most significant byte.</p></td>
<td></td>
</tr>
<tr class="odd">
<td><p>IpNetmask</p></td>
<td><p>Net::IpNetmask</p></td>
<td><p>A string representation of an IpAddress followed by either &quot;/n&quot; where n is a decimal value from 0 to 32 or &quot;/e.f.g.h&quot; where e-h are integer values between 0 and 255 and where when represented as binary from left to right the number is all 1s and the all 0s. The ip and netmask can also be passed in as positional or keyword arguments where the ip is an integer as described in IpAddress, and the netmask is a decimal value from 0 to 32.</p></td>
<td></td>
</tr>
<tr class="even">
<td><p>IpWithPort</p></td>
<td><p>Net::IpWithPort</p></td>
<td><p>A string representation of an IpAddress followed by &quot;:p&quot; where p is a decimal value from 0 to 65535. The ip and port can also be passed in as positional or keyword arguments where the ip is an integer as described in IpAddress, and the port is a decimal value from 0 to 65535.</p></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Time</p></td>
<td><p>tm</p></td>
<td><p>May be a Python struct_time, int, long, datetime, or date, the string &quot;Now&quot; or &quot;Today&quot;, or a string parseable by Python's strptime with one of the following formats:</p>
<p><code>* &quot;%a %b %d %H:%M:%S %Z %Y&quot;</code><br />
<code>* &quot;%a %b %d %H:%M:%S %Z %Y&quot;</code><br />
<code>* &quot;%Y/%m/%d %H:%M:%S&quot;</code><br />
<code>* &quot;%Y/%m/%d %H:%M&quot;</code><br />
<code>* &quot;%Y/%m/%d&quot;</code><br />
<code>* &quot;%m/%d/%Y %H:%M:%S&quot;</code><br />
<code>* &quot;%m/%d/%Y %H:%M&quot;</code><br />
<code>* &quot;%m/%d/%Y&quot;</code><br />
<code>* &quot;%m/%d/%y %H:%M:%S&quot;</code><br />
<code>* &quot;%m/%d/%y %H:%M&quot;</code><br />
<code>* &quot;%m/%d/%y&quot;</code></p></td>
<td></td>
</tr>
<tr class="even">
<td><p>subclasses of Enum</p></td>
<td><p>enum named after the parameter type</p></td>
<td><p>A string defined as part of the enum</p></td>
<td><p>This description applies to all enum parameter types which are defined as subclasses of Enum. The possible string values and optionally their mappings are specified in a dict called &quot;map&quot; or a list called &quot;vals&quot; defined as members of the Enum subclass itself.</p></td>
</tr>
<tr class="odd">
<td><p>Latency</p></td>
<td><p>Tick</p></td>
<td><p>Can be assigned an existing Clock or Frequency parameter, or a string with the format [value][unit] where value is a base 10 number and unit is one of the following:</p>
<p><code>* t =&gt; Ticks</code><br />
<code>* ps =&gt; picoseconds</code><br />
<code>* ns =&gt; nanoseconds</code><br />
<code>* us =&gt; microseconds</code><br />
<code>* ms =&gt; milliseconds</code><br />
<code>* s =&gt; seconds</code></p></td>
<td></td>
</tr>
<tr class="even">
<td><p>Frequency</p></td>
<td><p>Tick</p></td>
<td><p>Can be assigned an existing Latency or Clock parameter, or a string with the format [value][unit] where value is a base 10 number and unit is one of the following:</p>
<p><code>* THz =&gt; terahertz</code><br />
<code>* GHz =&gt; gigahertz</code><br />
<code>* MHz =&gt; megahertz</code><br />
<code>* kHz =&gt; kilohertz</code><br />
<code>* Hz =&gt; hertz</code></p></td>
<td><p>The frequency value is converted into a period in units of Ticks when transfered to C++.</p></td>
</tr>
<tr class="odd">
<td><p>Clock</p></td>
<td><p>Tick</p></td>
<td><p>Can be assigned an existing Latency or Frequency parameter, or a string with the format [value][unit] where value is a base 10 number and unit is one of the following:</p>
<p><code>* t =&gt; Ticks</code><br />
<code>* ps =&gt; picoseconds</code><br />
<code>* ns =&gt; nanoseconds</code><br />
<code>* us =&gt; microseconds</code><br />
<code>* ms =&gt; milliseconds</code><br />
<code>* s =&gt; seconds</code><br />
<code>* THz =&gt; terahertz</code><br />
<code>* GHz =&gt; gigahertz</code><br />
<code>* MHz =&gt; megahertz</code><br />
<code>* kHz =&gt; kilohertz</code><br />
<code>* Hz =&gt; hertz</code></p></td>
<td><p>This type is like a combination of the Frequency and Latency types described above.</p></td>
</tr>
<tr class="even">
<td><p>NetworkBandwidth</p></td>
<td><p>float</p></td>
<td><p>A floating point value specifying bits per second, or a string formatted as [value][unit] where value is a base 10 number and unit is one of the following:</p>
<p><code>* Tbps =&gt; terabits per second</code><br />
<code>* Gbps =&gt; gigabits per second</code><br />
<code>* Mbps =&gt; megabits per second</code><br />
<code>* kbps =&gt; kilobits per second</code><br />
<code>* bps =&gt; bits per second</code></p></td>
<td><p>The network bandwidth value is converted to Ticks per byte before being transfered to C++.</p></td>
</tr>
<tr class="odd">
<td><p>MemoryBandwidth</p></td>
<td><p>float</p></td>
<td><p>A string formatted as [value][unit] where value is a base 10 number and unit is one of the following:</p>
<p><code>* PB/s =&gt; pebibytes per second</code><br />
<code>* TB/s =&gt; tebibytes per second</code><br />
<code>* GB/s =&gt; gibibytes per second</code><br />
<code>* MB/s =&gt; mebibytes per second</code><br />
<code>* kB/s =&gt; kibibytes per second</code><br />
<code>* B/s =&gt; bytes per second</code></p></td>
<td><p>The memory bandwidth value is converted to Ticks per byte before being transferred to C++. kibi, mebi, etc. are true powers of 2.</p></td>
</tr>
<tr class="even">
<td><p>subclass of SimObject</p></td>
<td><p>defined in subclass</p></td>
<td></td>
<td><p>These parameter types are for assigning one simobject to another as a parameter. The may be set to nothing using the special &quot;NULL&quot; python object.</p></td>
</tr>
</tbody>
</table>
