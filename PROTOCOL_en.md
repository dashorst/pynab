# Nadb protocol

[ [:fr:](PROTOCOL.md) | [:gb:](PROTOCOL_en.md) | [:us:](PROTOCOL_en.md) ]

nabd is a TCP / IP server and thus interfaces with service daemons. It listens on port 10543.
Each packet is on a line (CRLF), encoded in JSON. Each package includes a "typical" slot.

## State packages

Indication of the rabbit's condition. This packet is sent during connection and when there is a change of state.

Issuer: nabd

- `{" type ":" state "," state ": state}`

The slot "" state "` can be:

- `" asleep "`: the rabbit sleeps;
- `" idle "`: the rabbit is awake and displays the info;
- `" interactive "`: the rabbit is in interactive mode;
- `" playing "`: the rabbit plays a command.

## `Info` Packages

Modification of the visual animation of the rabbit, that is to say what it displays at rest (`` idle '' mode).

Issuers: services

- `{" type ":" info "," request_id ": request_id," info_id ": info_id," animation ": animation}`

The slot "" request_id "` is optional and is returned in the response.

The required slot "" info_id "`indicates the identification of the info. It is this sequence which is modified.`info_id` is a string.

The optional "animation" slot indicates visual animation. If it is absent, the info is deleted. If present, it is an object:

`{" tempo ": tempo," colors ": colors}`

`tempo` is in ms.
`colors` is a list for the colors of the LEDs:

`{" left ": color," center ": color," right ": color}`

All slots are optional (`{}` = all LEDs are off).
`color` can be:

- a number from 0 to 15 representing a value in the original palette (0 = black, 15 = orange)
- a text representing the color in HTML format ('#' followed by 3 bytes in hexa) or symbolic

## Packets `ears`

Modification of the position of the ears at rest (`` idle '' mode). The position of the ears in interactive mode can be changed with a package "command" `via a choreography. The "ears" type package is designed for wedding ear service.

Issuers: services

- `{" type ":" ears "," request_id ": request_id," left ": left_ear," right ": right_ear}`

The slot "" request_id "` is optional and is returned in the response.

The slots "left" and "right" are optional (only one is required), and both "left_ear`and" right_ear` are integers representing the position.

## `command` packages

Command to be executed by the rabbit.

Issuers: services

- `{" type ":" command "," request_id ": request_id," sequence ": sequence," expiration ": expiration_date," cancelable ": cancelable}`

The slot "" request_id "` is optional and is returned in the response.

The slot "expiration" is optional and indicates the expiration date of the order. The command is played when the rabbit is available (not asleep, not doing something else) and if the expiration date is not reached.

The slot "" sequence "`is required and`sequence` is a list of elements of the type:

`{" audio ": audio_list," choreography ": choreography}`

The "audio" and "choreography" slots are optional.

`audio_list` is a list of sounds to play.

Each sound can be:

- a list of resources, separated by ";", the first one found is the one that will be played.
  Each resource is a path to a sound like `" nabmastodon / communion.wav "`. The algorithm first tries in the sub-directory of each application corresponding to the current language of the rabbit (`" sounds / fr_FR "`) then in the directory "" sounds "`, the applications in the order of`" settings.py "`. If the resource ends with`" _ "`or`" _ .suffixe "`, the sound is chosen at random from the elements of the corresponding directory.

`choreography` can be:

- a list of resources to the choreographies on the same mechanism as the sounds, in the `choreographies` directories of the different applications.
- `" urn: x-chor: streaming "` for the streaming choreography with random palette.
- `" urn: x-chor: streaming: N "` for the choreography of streaming with palette N.
- `" data: application / x-nabaztag-mtl-choreography; base64, <BASE64> "` for a choreography provided in Base64

The choreography is played during the playback of the various audio files in the list and is interrupted at the end of the audio.
If no sound is played, the choreography is played until the end.
If the same choreography is indicated for the different sequences, it is not interrupted.

The `cancelable` slot is optional. By default, the order will be canceled by a click on the button. If `cancelable` is`false`, the command is not canceled by the button (the service must manage the button).

## Packets `message`

Messages to be broadcast by the rabbit.

Issuers: services

- `{" type ":" message "," request_id ": request_id," signature ": signature," body ": body," cancelable ": cancelable}`

The slot "" request_id "` is optional and is returned in the response.

The slot "expiration" is optional and indicates the expiration date of the order. The command is played when the rabbit is available (not asleep, not doing something else) and if the expiration date is not reached.

The `` signature '' slot is optional and is of the type:

`{" audio ": audio_list," choreography ": choreography}`

The `` body '' slot is required and is a list of elements of the type:

`{" audio ": audio_list," choreography ": choreography}`

The "audio" and "choreography" slots are optional.

`audio_list` is a list of sounds to play, as for the`"command"`packages.
`choreography` is a choreography, as for the packages" "command" `. However, if the choreography is not specified, then the streaming choreography is used.

The signature is played first, followed by the body of the message, then the signature is replayed.

The `cancelable` slot is optional. By default, the order will be canceled by a click on the button. If `cancelable` is`false`, the command is not canceled by the button (the service must manage the button).

## Packages `cancel`

Cancels a command that is being executed (or scheduled).

Issuers: services

- `{" type ":" cancel "," request_id ": request_id}`

The `` request_id "` slot is required and corresponds to the `` request_id "` slot of the order placed. Only works for commands and messages, not for other packages.

## `wakeup` packages

Wake up the rabbit.

Issuers: services

- `{" type ":" wakeup "," request_id ": request_id}`

The slot "" request_id "` is optional and is returned in the response.

## `Sleep` packages

Put the rabbit to sleep. The rabbit falls asleep as soon as all the commands are executed and it is in "idle" mode.

Issuers: services

- `{" type ":" sleep "," request_id ": request_id}`

The slot "" request_id "` is optional and is returned in the response.

## `Mode` packages

Issuers: services

Change the mode for a given service.

- `{" type ":" mode "," request_id ": request_id," mode ": mode," events ": events}`

The slot "" mode "` can be:

- `" idle "`
- `" interactive "`

The optional "events" slot is a list with:

- `" asr "`
- `" button "`
- `" ears "`

For the "idle" mode, if `" events "` is not specified, this is equivalent to the empty list: the service does not receive any event. If `" asr "`, `" button "` or `" ears "` are specified, the service receives the corresponding events when the rabbit is awake and is not in `` interactive '' mode with another service. By default, the mode is `" idle "`, without events.

In the "interactive" mode, the service takes control of the rabbit and receives the specified events. The rabbit stops displaying the info. Only one service can be in interactive mode. If not specified, the service receives all events. The other services do not receive events, the rabbit does not play the commands and does not fall asleep. The interactive mode ends when the service sends a packet "" mode "`with the mode "" idle "` (or when the connection is broken).

## Packages `asr_event`

Issuer: nabd

Signals to services that a voice command has been understood.
The "nlu" slot contains the details of the command included, from the NLU engine.
In particular, the "intention" slot contains the detected intention.

`{'type': 'asr_event', 'nlu': {'intention': intention}}`

## `ears_event` packages

Issuer: nabd

Means to the services that the ears have been moved.
In "idle" mode, nabd calculates the position by launching a detection and sends to the services (for ear matching).
The package then has this form:

- `{" type ":" ears_event "," left ": ear_left," right ": ear_right}`

In "interactive" mode, nabd sends the fact that the ear has moved.

The package then has this form:

- `{" type ":" ears_event "," ear ": ear}`

## `button_event` packages

Issuer: nabd

Signals to services that the button has been pressed. Is sent to services that request this type of event (`" idle "` / `" interactive "` mode).

- `{" type ":" button_event "," event ": event}`

The slot "" event "` can be:

- `" down "`
- `" up "`
- `" click "`
- `" double_click "`
- `" click_and_hold "`

## `Response` packages

Issuer: nabd

Response from nabd to a package from a service.

- `{" type ":" response "," request_id ": request_id," status ":" ok "}`
- `{" type ":" response "," request_id ": request_id," status ":" canceled "}`
- `{" type ":" response "," request_id ": request_id," status ":" expired "}`
- `{" type ":" response "," request_id ": request_id," status ":" error "," class ": class," message ": message}`

The status "" ok "`means that the info has been added or that the command has been executed or the mode changed. In the case of an order, this response is sent when the order is completed. Ditto for the package "sleep"`. The slot "" request_id "`, if present, takes the id provided in the request.

The status "canceled" means that the user has canceled the order with the button.

The status "expired" means that the order has expired.

The status "" error "`means an error in the protocol.`class`and` message` are strings.

## `gestalt`,`test` and `config-update` packages

Used internally for communication between the website and nabd.
