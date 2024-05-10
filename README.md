

# Tesla_M60_GPU_Cooler
Open source, esp8366 based, esphome + home assistant solution to Nvidia Tesla card cooling
# Intro
Having started a journey to try and hack together some cheap system that I might be able to run llm's using ollama on (my current headless server is a SFF and couldn't be upgraded) I came across the Nivida Tesla cards, these old cards are designed for server farms, have "loads" of VRAM and can be got for a pretty cheap price nowadays on ebay.

After having a look about and stumbling on [this github issue](https://github.com/ollama/ollama/issues/2250) it would appear that the Nvidia Tesla M60 should have support for [Ollama](https://ollama.com/), and being that I had seen that floating about on eBay, I pulled the trigger and purchased it.

The [M60](https://www.techpowerup.com/gpu-specs/tesla-m60.c2760) is a "dual GPU" card, meaning its basically 2 graphics cards in 1, each with 8GB of DDR4 ram, meaning 16GB all together, not bad for a card picked up for <£100...

One thing about these cards is that they have no active cooling, and looking around online there seemed to be a few solutions out there that people have come up with (I'm not the only person with this idea it would seem ;)) 

The first one I found was [this](https://www.thingiverse.com/thing:6038375), a 3d printable shroud that you can put on the end of the card and then use an old Dell radial fan - 
![enter image description here](https://cdn.thingiverse.com/assets/81/43/b7/08/b5/large_display_0d693aff-a07b-4d63-ab91-155a881d37fa.png)

I liked this idea, and I have a 3d printer, so ordered the fan off ebay £6, then started printing the shroud.....it didn't fit, this print was designed for an M40, and although the description said it should fit, it didn't seem to fit my M60, being it was a little wider, and the psu connector was in a different location, luckily, the original poster had shared the Fusion 360 file, so I remixed "hacked" it to fit the [M60](https://www.thingiverse.com/thing:6611626). (STL is on this repo), This design needs you to repin the fan and then it just connects to a fan header on the mobo, annoyingly, I didn't seem to have any spare.

The next project I came across, and one that I have based a lot of my logic on is the [Tesla Cooler](https://github.com/tesla-cooler) project, using a piPico and a number of smaller fans to cool the card, this time the project has extended the functionally by having a pico control the fans, allowing it to speed up/down depending on the temps, I liked this, but didn't like the fact that the temp was being "estimated" based on the calculated difference that the `nvidia smi` cli tool gave and the readings of his `ds18b20` temperature sensor connected to the side of the card.
![enter image description here](https://www.esologic.com/wp-content/uploads/2021/11/MG_5596-644x429.jpg)
I thought there must be a better way to get accurate temp readings from the box and get them into MQTT or Home Assistant, and again, someone else had the idea and created [Sensors2mqtt](https://github.com/koriwi/sensors2mqtt), scraping the nvidia smi, then posting the information to an MQTT server, as well as making entities in Home Assistant, exactly what I wanted, saved me some coding..

Having seen these 3 projects, I decided I would build my own as a mesh of the best items I saw in each.

 - 3d printed shroud that I have modified to fit my card
 - Dell fan purchased from ebay 
 - ESP3688 (D1 Mini and ESP-12e) as I have loads sitting about
	 - Design a PCB for ease of use in case
 - Use `nvidia smi` information for accurate readings
 - Use `ds18b20` as a fall back
	 - Option to switch between the 2 sensors
	 - Option to override either sensor and set speed
  - 
# The Idea

So, Having looked at the provided projects, this is my approach, and should be yours if you are copying/following this...

 - Use sensors2mqtt to push nvidia-smi data to my already present MQTT server
	 - This will publish readings into `sensors2mqtt` topic
 - Using esp3688 and [ESPHome](https://esphome.io/index.html) to create a configuration that can 
	 - Connect to wifi
	 - Read temperature from published sensor reading on MQTT server
	 - Read temperature from local ds18b20 sensor
	 - All values and controls be visable via [Home Assistant](https://www.home-assistant.io/)
	 - Have a basic/configurable "Fan Scaling" against the GPU temp
		 - This is set to -
			 - 25% <40°C
			 - 26%-80% =>40°C =<100°C
			 - 100% =>100°C

![image](https://github.com/karl0ss/Tesla_M60_GPU_Cooler/assets/2493260/5208d1f0-97dd-4c7e-82b3-e253288f31ae)



