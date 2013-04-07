Forward your VM's ports to access it from External Networks/Internet.
===================


Well to do port forwarding is a trivial two step task. 

* Go to virtual box , right click on your VM and click on settings.

* In the settings page go to Networks , select the adapter tab which is having NAT connection. As you can forward only NAT connections ports.

* Select the button `Port Forwarding`.

    .. image:: https://raw.github.com/dguitarbite/OpenStack-Folsom-VM-SandBox-Guide/VirtualBox/Images/ScreenShots/5.Port%20Forwarding/Port%20Forwarding%201.png

* Add a TCP/IP rule. And select the required ports. 

    .. image:: https://raw.github.com/dguitarbite/OpenStack-Folsom-VM-SandBox-Guide/VirtualBox/Images/ScreenShots/5.Port%20Forwarding/Port%20Forwarding%202.png

* The following table will give you a basic idea of the ports .

    +-------------------+--------------------------------------+------------------------------------------------------------------+--------------------------------------------------------------+---------------------------------------------------------------+
    |  Name             |  Protocol       |     Host Ip        | Host Port                                                        |       Guest IP                                               |    Guest Port                                                 |
    +===================+=================+====================+==================================================================+==============================================================+===============================================================+
    | Name of the Rule  | Protocl Name    | Can Be Left Blank  | Select Appropriate Host Port (This Port should not be under use) |Can be Left Blank                                             |  Your VM's Port Number (Usually port 80 should do the trick)  |
    +-------------------+--------------------------------------+------------------------------------------------------------------+--------------------------------------------------------------+---------------------------------------------------------------+
