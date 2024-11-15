## XR829 5.15 Mainline Driver

Patch the dtsi file of your soc if it does not contain `addr_mgt`, because this driver depends on sunxi_addr kernel module which needs this section:

```dts
addr_mgt: addr-mgt {
	compatible = "allwinner,sunxi-addr_mgt";
	type_addr_wifi = <0x2>;
	type_addr_bt = <0x2>;
	type_addr_eth = <0x2>;
	status = "okay";
};
```

And the board dts as this:
```dts
/ {

	/* your board definitions */

	wifi_pwrseq: wifi_pwrseq {
		compatible = "mmc-pwrseq-simple";
		reset-gpios = <&pio 0 8 GPIO_ACTIVE_LOW>; /* PA8 */
		// clocks = <&rtc CLK_OSC32K_FANOUT>;     /* clocks may not necessary? */
	};
}

&mmc1 {
	vmmc-supply = <&reg_vcc3v3>;
	vqmmc-supply = <&reg_vcc3v3>;
	mmc-pwrseq = <&wifi_pwrseq>;
	bus-width = <4>;
	non-removable;
	status = "okay";

	xr819: sdio_wifi@1 {
		reg = <1>;
	};
};
```

Copy the source folder to `drivers/net/wireless/xr829`,  
apply these options:
- CONFIG_XR829_WLAN=m
- CONFIG_XR829_SUSPEND_POWER_OFF=n

Please compile this driver as a module(aka: -m), it seems it does not support -y option.

Make sure the firmware files are under `/lib/firmware/xr829`. when the module loads, it search for the firmware within this path and downloads it into xr829.

## What works
This driver has been tested in sunxi 5.15.93 on an Allwinner-H3 board(Armbian bullseye). **WLAN works, but BT doesn't.**
