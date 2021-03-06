# FHEM-Tado
A FHEM extension to interact with Tado cloud

The FHEM extension requires two files:
 - 98_Tado.pm
 - 98_TadoDevice.pm

The extension is build based on the two-tier module concept.
The 98_Tado.pm contains the source code to define a Tado device. The Tado device acts like a bridge and manages the communication towards the Tado cloud.
The 98_TadoDevice.pm contains the code to define the different TadoDevices. The devices do represent either a physical device represented by a serial number, a so called zone basically representing a room or the weather channel containing the weather data used by Tado to optimize the heating times.

<h2>There are still some things todo which I have not done so far:</h2>
<ul>
<li>- Parse the several date information and bring this to local time</li>
<li>- Add validation on inserted serial numbers</li>
</ul>


<h3>Tado</h3>
<ul>
    <i>Tado</i> implements an interface to the Tado cloud. The plugin can be used to read and write
    temperature and settings from or to the Tado cloud. The communication is based on the reengineering of the protocol done by
    Stephen C. Phillips. See <a href="http://blog.scphillips.com/posts/2017/01/the-tado-api-v2/">his blog</a> for more details.
    Not all functions are implemented within this FHEM extension. By now the plugin is capable to
    interact with the so called zones (rooms) and the registered devices. The devices cannot be
    controlled directly. All interaction - like setting a temperature - must be done via the zone and not the device.
    This means all configuration like the registration of new devices or the assignment of a device to a room
    must be done using the Tado app or Tado website directly. Once the configuration is completed this plugin can
    be used.
    This device is the 'bridge device' like a HueBridge or a CUL. Per zone or device a dedicated device of type
    'TadoDevice' will be created.
    <br><br>
    <a name="Tadodefine"></a>
    <b>Define</b>
    <ul>
        <code>define &lt;name&gt; Tado &lt;username&gt; &lt;password&gt; &lt;interval&gt;</code>
        <br><br>
        Example: <code>define TadoBridge Tado mail@provider.com somepassword 120</code>
        <br><br>
        The username and password must match the username and password used on the Tado website.
        Please be aware that username and password are stored and send as plain text. They are visible in FHEM user interface.
        It is recommended to create a dedicated user account for the FHEM integration.
        The Tado extension needs to pull the data from the Tado website. The 'Interval' value defines how often the value is refreshed.
    </ul>
    <br>
    <b>Set</b><br>
    <ul>
        <code>set &lt;name&gt; &lt;option&gt;</code>
        <br><br>
        The <i>set</i> command just offers very limited options.
        If can be used to control the refresh mechanism. The plugin only evaluates
        the command. Any additional information is ignored.
        <br><br>
        Options:
        <ul>
              <li><i>interval</i><br>
                  Sets how often the values shall be refreshed.
                  This setting overwrites the value set during define.</li>
              <li><i>start</i><br>
                  (Re)starts the automatic refresh.
                  Refresh is autostarted on define but can be stopped using stop command. Using the start command FHEM will start polling again.</li>
              <li><i>stop</i><br>
                  Stops the automatic polling used to refresh all values.</li>
        </ul>
    </ul>
    <br>
    <a name="Tadoget"></a>
    <b>Get</b><br>
    <ul>
        <code>get &lt;name&gt; &lt;option&gt;</code>
        <br><br>
        You can <i>get</i> the major information from the Tado cloud.
 	    	<br><br>
        Options:
        <ul>
              <li><i>home</i><br>
                  Gets the home identifier from Tado cloud.
                  The home identifier is required for all further actions towards the Tado cloud.
                  Currently the FHEM extension only supports a single home. If you have more than one home only the first home is loaded.
                  <br/><b>This function is automatically executed once when a new Tado device is defined.</b></li>
              <li><i>zones</i><br>
                  Every zone in the Tado cloud represents a room.
                  This command gets all zones defined for the current home.
                  Per zone a new FHEM device is created. The device can be used to display and
                  overwrite the current temperatures.
                  This command can always be executed to update the list of defined zones. It will not touch any existing
                  zone but add new zones added since last update.
                  <br/><b>This function is automatically executed once when a new Tado device is defined.</b></li>
                  </li>
              <li><i>update</i><br/>
                      Updates the values of: <br/>
                      <ul>
                         <li>All Tado zones</li>
                         <li>All mobile devices - if attribute <i>generateMobileDevices</i> is set to true</li>
                         <li>The weather device - if attribute <i>generateWeather</i> is set to true</li>
                      </ul>
                      This command triggers a single update not a continuous refresh of the values.  
                  </li>              
              <li><i>devices</i><br/>
                  Fetches all devices from Tado cloud and creates one TadoDevice instance
                  per fetched device. This command will only be executed if the attribute <i>generateDevices</i> is set to <i>yes</i>. If the attribute is set to <i>no</i> or not existing an error message will be displayed and no communication towards Tado will be done.
                  This command can always be executed to update the list of defined devices.
                  It will not touch existing devices but add new ones.
                  Devices will not be updated automatically as there are no values continuously changing.
                  </li>
              <li><i>mobile_devices</i><br/>
                  Fetches all defined mobile devices from Tado cloud and creates one TadoDevice instance
                  per mobile device. This command will only be executed if the attribute <i>generateMobileDevices</i> is set to <i>yes</i>. If the attribute is set to <i>no</i> or not existing an error message will be displayed and no communication towards Tado will be done.
                  This command can always be executed to update the list of defined mobile devices.
                  It will not touch existing devices but add new ones.
              </li>
              <li><i>weather</i><br/>
                  Creates or updates an additional device for the data bridge containing the weather data provided by Tado. This command will only be executed if the attribute <i>generateWeather</i> is set to <i>yes</i>. If the attribute is set to <i>no</i> or not existing an error message will be displayed and no communication towards Tado will be done.
              </li>
        </ul>              
    </ul>
    <br>   
    <a name="Tadoattr"></a>
    <b>Attributes</b>
    <ul>
        <code>attr &lt;name&gt; &lt;attribute&gt; &lt;value&gt;</code>
        <br><br>
         You can change the behaviour of the Tado Device.
         <br><br>
         Attributes:
         <ul>
               <li><i>generateDevices</i><br>
                   By default the devices are not fetched and displayed in FHEM as they don't offer much functionality.
                   The functionality is handled by the zones not by the devices. But the devices offers an identification function <i>sayHi</i> to show a message on the specific display. If this function is required the Devices can be generated. Therefor the attribute <i>generateDevices</i> must be set to <i>yes</i>
                   <br/><b>If this attribute is set to <i>no</i> or if the attribute is not existing no devices will be generated..</b>
                </li>
                <li><i>generateMobileDevices</i><br>
                    By default the mobile devices are not fetched and displayed in FHEM as most users already have a person home recognition. If Tado shall be used to identify if a mobile device is at home this can be done using the mobile devices. In this case the mobile devices can be generated. Therefor the attribute <i>generateMobileDevices</i> must be set to <i>yes</i>
                    <br/><b>If this attribute is set to <i>no</i> or if the attribute is not existing no mobile devices will be generated..</b>
                 </li>
                 <li><i>generateWeather</i><br>
                     By default no weather channel is generated. If you want to use the weather as it is defined by the tado system for your specific environment you must set this attribute. If the attribute <i>generateWeather</i> is set to <i>yes</i> an additional weather channel can be generated.
                     <br/><b>If this attribute is set to <i>no</i> or if the attribute is not existing no Devices will be generated..</b>
                  </li>
          </ul>
    </ul>
</ul>



<h3>TadoDevice</h3>
<ul>
    <i>TadoDevice</i> is the implementation of a zone, a device, a mobile device or the weather channel connected to one Tado instance and therefor one tado cloud account.
    It can only be used in conjunction with a Tado instance (a Tado bridge).
    The TadoDevice is intended to display the current measurements of a zone or device and allow
    the interaction. It can be used to set or reset the temperature within a zone or to
    display a "hi" statement on a physical Tado device. It can also be used to identify which mobile devices are at home.
    TadoDevices should not be created manually. They are auto generated once a Tado device is defined.
    <br><br>
    <a name="TadoDevicedefine"></a>
    <b>Define</b>
    <ul>
        <code>define &lt;name&gt; TadoDevice &lt;TadoId&gt; &lt;IODev=IODeviceId&gt;</code>
        <br><br>
        Example: <code>define kitchen TadoDevice 1 IODev=TadoBridge</code>
        <br><br>
        Normally the define statement should be called by the Tado device.
        If called manually the TadoId and the IO-Device must be provided.
        The TadoId is either the zone Id if a zone shall be created or the serial number
        of a physical device. The IO-Device must be of type Tado.
    </ul>
    <br>
    <a name="TadoDeviceset"></a>
    <b>Set</b><br>
    <ul>
        <code>set &lt;name&gt; &lt;option&gt; &lt;value&gt;</code>
        <br><br>
        What can be done with the <i>set</i> command is depending on the subtype
        of the TadoDevice. For all thermostats it is possible to set the temperature using the
        automatic, temperature and temperature-for options. For all physical devices the
        sayHi option is available.
        <br><br>
        Options:
        <ul>
              <li><i>sayHi</i><br>
                  <b>Works on devices only.</b>
                  Sends a request to the a specific physical device. Once the request
                  reaches the device the device displays "HI".
                  Command can be used to identify a physical device.
                  </li>
              <li><i>automatic</i><br>
                  <b>Works on zones only.</b>
                  Resets all temperature settings for a zone.
                  The plan defined in the cloud (either by app or browser) will be used to set the temperature</li>
              <li><i>off</i><br>
                  <b>Works on zones only.</b>
                  Turns the heating in the specific zone completely off.
                  The setting will be kept until a new temperature is defined via app, browser or FHEM.
              <li><i>temperature</i><br>
                  <b>Works on zones only.</b>
                  Sets the temperature for a zone.
                  The setting will be kept until a new temperature is defined via app, browser or FHEM.
                  Value can be <i>off</i> or any numeric value between 4.0 and 25.0 with a precision of 0.1 degree.
              <li><i>temperature-for-60</i><br>
                  <b>Works on zones only.</b>
                  Sets the temperature for a zone for 60 minutes only.
                  The temperature will be kept for 60 minutes. Afterwards the zone will fall back to the standard plan defined in app or web. Value definition is like for command <i>set temperature</i>.</li>
             <li><i>temperature-for-90</i><br>
                  <b>Works on zones only.</b>
                   Sets the temperature for a zone for 90 minutes only.
                  The temperature will be kept for 90 minutes. Afterwards the zone will fall back to the standard plan defined in app or web. Value definition is like for command <i>set temperature</i>.</li>
             <li><i>temperature-for-120</i><br>
                  <b>Works on zones only.</b>
                  Sets the temperature for a zone for 120 minutes only.
                  The temperature will be kept for 120 minutes. Afterwards the zone will fall back to the standard plan defined in app or web. Value definition is like for command <i>set temperature</i>.</li>
              <li><i>temperature-for-180</i><br>
                  <b>Works on zones only.</b>
                  Sets the temperature for a zone for 180 minutes only.
                  The temperature will be kept for 180 minutes. Afterwards the zone will fall back to the standard plan defined in app or web. Value definition is like for command <i>set temperature</i>.</li>
             <li><i>temperature-for-240</i><br>
                  <b>Works on zones only.</b>
                   Sets the temperature for a zone for 240 minutes only.
                  The temperature will be kept for 240 minutes. Afterwards the zone will fall back to the standard plan defined in app or web. Value definition is like for command <i>set temperature</i>.</li>
             <li><i>temperature-for-300</i><br>
                  <b>Works on zones only.</b>
                  Sets the temperature for a zone for 300 minutes only.
                  The temperature will be kept for 300 minutes. Afterwards the zone will fall back to the standard plan defined in app or web. Value definition is like for command <i>set temperature</i>.</li>                    
        </ul>
    </ul>
    <br>
    <a name="TadoDeviceget"></a>
    <b>Get</b><br>
    <ul>
        <code>get &lt;name&gt; &lt;option&gt;</code>
        <br><br>
        The only available <i>get</i> function is called <b>update</b> and can be used to update all readings of the specific TadoDevice.
        <br><br>
        Options:
        <ul>
            <li><i>Update</i><br>
            <b>This <i>get</i> command is available on zones, weather and mobile devices</b>
            This call updates the readings of a <i>TadoDevie</i> with the latest values available in the Tado cloud.
            </li>
        </ul>
    </ul>
    <br>    
    <a name="TadoDeviceattr"></a>
    <b>Attributes</b>
    <ul>
        <code>attr &lt;name&gt; &lt;attribute&gt; &lt;value&gt;</code>
        <br><br>
         There is one attribute available. It only affects zones.
        <br><br>
        Attributes:
        <ul>
            <li><i>earlyStart</i> true|false<br>
                When set to true the Tado system starts to heat up before the set heating change.
                The intention is to reach the target temperature right at the point in time defined.
                E.g. if you want to change the temperature from 20 degree to 22 degree on 6pm early start
                would start heating at 5:30pm so the zone is on 22 degree at 6pm.
                The early start is a feature of Tado. How this is calculated and how early the heating is started
                is up to Tado.
            </li>
        </ul>
    </ul>
</ul>
