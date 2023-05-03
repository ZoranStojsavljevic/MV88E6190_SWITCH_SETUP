### Setting the Marvell mv88e6190 switch with i.MX6 via rgmii interface [ETH/BR layer]

#### [1] Adding TxC and RxC clock skew

Please, do read the following page to get familiar with some required setups for the mv88e6190.
https://ethernetfmc.com/rgmii-interface-timing-considerations/

#### [2] Device Tree Source
```
&fec {
	pinctrl-names = "default";
	/* pinctrl-0 = <&pinctrl_enet>; */
	pinctrl-0 = <&pinctrl_enet_5>;

	/*
	 * Instead phy-mode "rgmii" the "rgmii-id" mode is entered, because
	 * i.MX6 silicon has the silicon bug, and it is not able to impose
	 * the required delay (clock skew) on TxC and RxC rgmii lines. Given
	 * mode ("rgmii-id") is instructing the DSA driver to insert these
	 * two delays on port 0 (MAC to MAC management port) mv88e6190.
	 */

	phy-mode = "rgmii-id";
	local-mac-address = [XX XX XX XX XX XX];
	/* fsl,err006687-workaround-present; */
	status = "okay";

	fixed-link {
		speed = <1000>;
		full-duplex;
	};

	mdio {
		#address-cells = <1>;
		#size-cells = <0>;
		status = "okay";

		switch0: switch0@0 {
			compatible = "marvell,mv88e6190";
			pinctrl-names = "default";
			pinctrl-0 = <&pinctrl_switch>;
			reg = <0>;
			dsa,member = <0 0>;
			reset-gpios = <&gpio1 29 GPIO_ACTIVE_HIGH>;

			ports {
				#address-cells = <1>;
				#size-cells = <0>;

				port@0 {
					reg = <0>;
					label = "cpu";
					ethernet = <&fec>;
					phy-mode = "rgmii-id";

					fixed-link {
						speed = <1000>;
						full-duplex;
					};
				};

				port@1 {
					reg = <1>;
					label = "lan1";
				};

				port@2 {
					reg = <2>;
					label = "lan2";
				};


				port@3 {
					reg = <3>;
					label = "lan3";
				};

				port@4 {
					reg = <4>;
					label = "lan4";
				};

				port@5 {
					reg = <5>;
					label = "lan5";
				};
			};
		};
	};
};
```
#### [3] Setting the kernel.org vanilla kernel to integrate the mv88e6190 switch into the kernel

	Switch (and switch-ish) device support @ Networking support->Networking options
	Distributed Switch Architecture @ Networking support->Networking options
	Tag driver for Marvell switches using DSA headers @ Networking	support->Networking options->Distributed Switch Architecture
	Tag driver for Marvell switches using EtherType DSA headers @ Networking support->Networking options->Distributed Switch Architecture
	Marvell 88E6xxx Ethernet switch fabric support @ Device	Drivers->Network device support->Distributed Switch Architecture drivers
	Switch Global 2 Registers support @ Device Drivers->Network device support->Distributed Switch Architecture drivers->Marvell 88E6xxx Ethernet switch fabric support
	Freescale devices @ Device Drivers->Network device support->Ethernet driver support
	FEC ethernet controller (of ColdFire and some i.MX CPUs) @ Device Drivers->Network device support->Ethernet driver support->Freescale devices
	Marvell devices @ Device Drivers->Network device support->Ethernet driver support
	Marvell MDIO interface support @ Device Drivers->Network device	support->Ethernet driver support->Marvell devices MDIO Bus/PHY emulation with fixed speed/link PHYs @ Device
	Drivers->Network device support->PHY Device support and infrastructure

#### [4] Configure the switch to be a bridge

	ip link set eth0 up
	ip link set lan1 up
	ip link set lan2 up
	ip link set lan3 up
	ip link set lan4 up
	ip link set lan5 up
	ip link name br0 type bridge
	ip link set br0 up
	ip link lan1 master br0
	ip link lan2 master br0
	ip link lan3 master br0
	ip link lan4 master br0
	ip link lan5 master br0
	ip addr add 192.168.1.4/24 dev br0

#### [5] References: DSA driver kernel extension for dsa mv88e6190 switch

	https://www.spinics.net/lists/netdev/msg600586.html (by Zoran Stojsavljevic)
	https://www.spinics.net/lists/netdev/msg600590.html (by Andrew Lunn)
	https://www.spinics.net/lists/netdev/msg600985.html (by Zoran Stojsavljevic)
	https://www.spinics.net/lists/netdev/msg600987.html (by Andrew Lunn)
	https://www.spinics.net/lists/netdev/msg601303.html (by Zoran Stojsavljevic)
	https://www.spinics.net/lists/netdev/msg601325.html (by Andrew Lunn)
	https://www.spinics.net/lists/netdev/msg601368.html (by Zoran Stojsavljevic)
	https://www.spinics.net/lists/netdev/msg601390.html (by Andrew Lunn)
	https://www.spinics.net/lists/netdev/msg601403.html (by Zoran Stojsavljevic)

#### [6] IMPORTANT NOTE to Marvell mv88exxxx users (about Marvell DSA bug)

The Marvell DSA (Linux Distributed Switch Architecture) switch bugs for DSA were solved in the Linux kernel 5.15.x, since the patches for the bugs in the Marvell DSA driver were ported to the kernel 5.15.x releases, but due to complexity and too many changes in the DSA architecture these changes were not back ported to earlier than 5.15.x kernels.

All of this was tested on a Linksys WRT3200ACMv1 device running OpenWRT SNAPSHOT r22702-cf8d861978 with Linux kernel version 5.15.109.
