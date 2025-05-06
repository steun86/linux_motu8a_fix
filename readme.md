# Create virtual sink for Motu 8a

This creates a virtual audio sink called virtual_sink_motu which captures application output and routes it directly to AUX0/AUX1 of the MOTU 8A interface. It's useful for games or applications (like Steam/Proton) that don't support multichannel routing directly.

## 1. Get the name of the audio device

```bash
pw-dump | grep '"node.name"' | grep -i motu
```

This returned:

```bash
"node.name": "alsa_output.usb-MOTU_8A_0001f2fffe00a1a9-00.multichannel-output"
```

## 2. Create a shell script that runs the loopback

```bash
mkdir -p ~/.local/bin
nano ~/.local/bin/create-virtual-sink
```

Paste this:

```bash
#!/bin/bash

exec pw-loopback \
--capture-props='node.name=virtual_sink_motu,media.class=Audio/Sink,node.description=Virtual Sink to MOTU AUX,audio.position=[FL FR]' \
--playback-props='target.object=alsa_output.usb-MOTU_8A_0001f2fffe00a1a9-00.multichannel-output,audio.position=[AUX0 AUX1],stream.dont-remix=true,node.passive=true'
```

Save and make it executable:

```bash
chmod +x ~/.local/bin/create-virtual-sink
```

## 3. Create a systemd user unit

```bash
mkdir -p ~/.config/systemd/user
nano ~/.config/systemd/user/virtual-sink-motu.service
```

Paste this:

```bash
[Unit]
Description=Create virtual sink to MOTU AUX0/AUX1
After=pipewire.service

[Service]
ExecStart=%h/.local/bin/create-virtual-sink
Restart=on-failure

[Install]
WantedBy=default.target
```

## 4. Enable and start

```bash
systemctl --user daemon-reexec
systemctl --user daemon-reload
systemctl --user enable --now virtual-sink-motu.service
```

## Done! Now check:

```bash
pactl list short sinks
```

You should see:

```bash
virtual_sink_motu
```

And it routes audio to AUX0/AUX1 on your MOTU.

You can restart the service manually to be sure it's running:

```bash
systemctl --user restart virtual-sink-motu.service
```

## Fix: No audio in Steam with Proton

Proton will output audio to whatever is set as your default sink in PulseAudio (via PipeWire).

You can set this in terminal:

```bash
pactl set-default-sink virtual_sink_motu
```
