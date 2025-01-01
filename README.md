# Android Telemarketer Deterrent

## Introduction:

The inspiration for this came out of the frustration of receiving multiple calls, day in and day out, generally all of them pertaining to same thing, all of them from different numbers from every area code of my state.  This went on for a few months, all while using the TrueCaller app, until finally I had enough of it.

Is this overkill?  Absolutely!  But maybe I'll get some peace and quiet, and maybe you can too.

&nbsp;

## High-Level Overview:

1.  Non-contact callers will receive instructions on how to proceed.
2.  Callers that do not press a button that have a >= 40% spam score go to voicemail.
3.  All other callers will ring back to you, but can be declined and sent to voicemail.
4.  Voicemail transcriptions are texted to you, with a link to play the voicemail audio.

&nbsp;

## Requirements:

- Android Phone (only because of the following app I'm using)
- Android App: [Spam Call Blocker for Android (RoboBlocker)](https://play.google.com/store/apps/details?id=me.spam.robo.call.blocker.call.filter.app)
    - Free version is all you need.
- [Twilio](https://www.twilio.com/try-twilio) Account (you can also signup with your Google account)
    - You receive a $15 USD credit at the time of this writing.
    - Up-front Twilio expenses (prices may vary):
        - Phone Numbers (Capable of Voice and SMS/MMS texting):
            - Toll-Free Phone Number: Free initially, then $2.15 USD / month
            - Local Phone Number: $1.15 USD / month
        - Messaging (SMS/MMS) for a Toll-Free Phone Number:
            - Must verify, but Free.
        - Messaging (SMS/MMS) A2P Registration Fees for a Local Phone Number:
            - One-Time: $15 USD
            - Monthly: $2 USD
    - Other Twilio expenses:
        - Outbound SMS:
            - Standard Outbound SMS Segment pricing: $0.0079 USD / message segment
            - SMS Failed Message Processing Fee: $0.001 USD / message
            - SMS Carrier Fees: $0.003 USD / message
        - Programmable Voice: $0.017 USD / minute
        - Caller Name Lookups: $0.01 USD / lookup
        - Text-To-Speech - Amazon Polly: $0.0008 USD / use
        - Voice Recordings: $0.0025 USD / minute
        - Transcriptions: $0.05 USD / transcription
            - *For cheaper and potentially more accurate transcriptions, you might try* [Deepgram](https://deepgram.com/) *for AI transcriptions.  Their Pay As You Go plan includes $200 USD credit, and then their Pre-Recorded Pay As You Go pricing is anywhere from $0.0043/min to $0.0145/min.  I've not used them personally, not yet anyways, nor do I currently include instructions for integrating them into this setup.*
        - Add-On: IceHook Systems Scout: $0.0035 USD / caller lookup (for checking caller spam score)
        - Minimum dollar amount for adding funds or upgrading your Twilio account: $20 USD

&nbsp;

## Setting up Spam Call Blocker for Android (RoboBlocker):

Once RoboBlocker is installed, open it and configure it like so.

Tap on the RoboBlocker "tab" at the bottom center of the RoboBlocker app.

Under Settings, from the top:

- Pause Blocking: Disabled (Maybe Enable this for now, until you've setup everything.)
- Block Spam Call (Best Value): OFF
- Show Caller ID (Free): OFF  (Turn it on if you want.)
- Contacts Only (Free): ON  (This is important!)
- Block & Allow List: Empty for now (You can add callers here accordingly.)
- AI Assistant: OFF  (This is important!)

Tap on "More Block Settings" at the bottom and set these options:

- Allow Repeated Calls: ON
    - This is important for 2 reasons:
        1.  If someone needs you urgently from an unusual number, such as a school, hospital, or stranger's phone.
        2.  The call flow for this setup relies on it.
- The rest is mostly up to you, but these are my preferences:
    - Block International Calls: ON
    - Block Neighbor-Spoofing Calls: ON
    - Anonymous Calls Rejection: ON
    - Toll-Free Calls: OFF
    - Fight Back: OFF  (This requires turning on the AI Assistant.  Do not use.)

&nbsp;

## Setting up Twilio

See the following guide to create your Twilio account and get started:

- Inside the US and Canada: https://www.twilio.com/docs/usage/tutorials/how-to-use-your-free-trial-account-namer
- \*Outside of the US and Canada: https://www.twilio.com/docs/messaging/guides/how-to-use-your-free-trial-account

\*Please note that this guide primarily geared towards US users, so some of this may differ for non-US and non-Canadian users.

\--> Please note, as part of the above guides, you will need to either verify your Toll-Free Number to enable outbound text messaging (verify for free; note that your Toll-Free Number does cost $2.15 USD / month after your first month), or if you opt upgrade your account (for $20 USD), you can spend your account funds on a local phone number ($1.15 USD / month) and A2P Messaging Registration to enable outbound text messaging (One-Time Fee of $17 USD + $2 USD / month).  Also note that this does not include the Standard Outbound SMS Segment pricing of $0.0079 USD / message segment + Outbound SMS Carrier Fee of $0.003 USD / message.  See the Requirements section (above) for more pricing details, or refer to the Twilio website for current pricing details.

Once logged into Twilio, you'll have 2 tabs at the top of the left-side pane:

- Develop: This is where all the setup happens.
- Monitor: In here, among other things, you can review:
    - Errors (Errors -> Error logs and Errors -> Webhook)
    - Calls (Logs -> Calls)
    - Voicemails (Logs -> Call recordings)
    - Voicemail Transcriptions (Logs -> Call transcriptions)
    - Text Messages (Logs -> Messaging)

&nbsp;

We'll be making use of a few Twilio products and features, specifically:

- Phone Numbers (for receiving our calls that are initially blocked by RoboBlocker and for sending voicemail notifications from)
- Messaging (for sending text message notifications of voicemails left at Twilio: this includes a transcription and a link to the audio)
- Studio (for managing the call flow when calls come into the Twilio phone number)
- Sync Documents (for preventing call flow loops and sending the caller to Twilio voicemail if you should decline or miss the incoming call)
- Functions and Assets (for texting voicemail notifications and providing a voicemail playback link)

&nbsp;

### Setting up Twilio: Functions and Assets

1.  Scroll to the bottom of the left-side pane (under the Develop tab), and click on "Explore Products +".  Click on Developer Tools or scroll down to Developer Tools, and click the pin next to Functions and Assets.  Now, Functions and Assets will appear at the bottom of the left-side pane.
    
2.  Click on Functions and Assets in the left-side pane, and choose Services.  Click the Create Service button, and enter a Service Name of "voicemail" and click Next.  Now you're inside the Service editor.
    
3.  Click on Dependencies under Settings & More at the bottom left.
    
    1.  Ensure that Node Version is set to "Node.js v18".
    2.  Add the following dependency: Module: request, Version: 2.8.22
4.  Click on Environment Variables under Settings & More at the bottom left.
    
    1.  Ensure that "Add my Twilio Credentials (ACCOUNT_SID) and (AUTH_TOKEN) to ENV" is checked.
    2.  Add the following environment variables:
        1.  Key: TOKEN, Value: &lt;short set of random numbers and letters&gt;
        2.  Key: VM_TXT_FROM, Value: &lt;Twilio Phone Number&gt;  (ie, +18885551234)
        3.  Key: VM_TXT_TO, Value: &lt;Your Phone Number&gt; (same format: +14012224567)
5.  Click on the blue "Add +" button at the top left, and then click Add Function.  We will create the following 2 functions:
    
    1.  For the first function, change the path to "/t" (t for text message), press Enter to create the function, and ensure the icon to the right is a padlock and key, making it a "Protected" function.  Next, fill in the following code for this function:
        
        ```JavaScript
        exports.handler = async function(context, event, callback) {
          const client = context.getTwilioClient();
        
          const token = process.env.TOKEN;
          const vm_from = event.f.replace(/^[^0-9]*1([0-9]{3})([0-9]{3})([0-9]{4})$/, "$1-$2-$3");
          const rec_sid = event.RecordingSid;
          const rec_url = `https://${process.env.DOMAIN_NAME}/v?t=${token}&s=${rec_sid}`;
          const transcriptions_url = event.TranscriptionUrl + ".json";
          
          const response = await fetch(transcriptions_url, {method: 'GET', headers: { 'Authorization': 'Basic ' + Buffer.from(`${process.env.ACCOUNT_SID}:${process.env.AUTH_TOKEN}`).toString('base64')}});
          const data = await response.json();
        
          var from = process.env.VM_TXT_FROM;
          const to = process.env.VM_TXT_TO;
          const body = `Voicemail from ${vm_from}
        Transcription: ${data.transcription_text}
        Voicemail audio: ${rec_url}`;
        
          await client.messages.create({to, from, body});
        
          return callback(null, null);
        };
        ```
        
    2.  For the second function, change the path to "/v" (v for voicemail audio), press Enter to create the function, and ensure the icon to the right is a globe with "www" written on it (click on the icon to change it), making it a "Public" function.  Next, fill in the following code for this function:
        
        ```JavaScript
        exports.handler = function(context, event, callback) {
          const received_token = event.t;
          const received_rec_sid = event.s;
        
          if (received_token != process.env.TOKEN) {
            return callback(null, null);
          }
        
          const request = require('request');
          const twresponse = new Twilio.Response();
        
          return request({method: 'GET', uri: `https://${process.env.ACCOUNT_SID}:${process.env.AUTH_TOKEN}@api.twilio.com/2010-04-01/Accounts/${process.env.ACCOUNT_SID}/Recordings/${received_rec_sid}.wav`, encoding: null}, function (error, response, body) {
            twresponse.setBody(body);
            twresponse.appendHeader('Content-Type', 'audio/x-wav');
            return callback(null, twresponse);
          });
        };
        ```
        
6.  Click on the blue Deploy All button at the bottom-left of the editor.
    

&nbsp;

### Setting up Twilio: IceHook Scout Add-On

1.  Click on Marketplace from the left-side pane under the Develop tab, and choose Catalog.
2.  Underneath Filter By Products, choose Lookup.
3.  Next, find IceHook Systems Scout and click on it.
4.  Click the blue "+ Install" button.
5.  Read and agree to the terms and conditions, then click Install.
6.  Underneath the Configure tab, leave the Unique Name alone ("icehook_scout"), place a checkmark next to Incoming Voice Call underneath USE IN, and click Save.

&nbsp;

### Setting up Twilio: Studio Flow

1.  Click on Studio in the left-side pane of the Develop tab, and choose Flows.
2.  Click on the blue Create new Flow button at the top right and give the new flow a name, such as Telemarketer Deterrent, and click Next.
3.  Scroll to the bottom of the list and choose Import from JSON and click Next.
4.  Clear the JSON text box, paste in the following JSON, replace the following placeholders, click Next, and then click the red Publish button at the top of the Studio Flow editor.  Now click the blue Publish button if prompted.
    1.  ACCOUNT_SID -- replace this with your Account SID, which can be found on your Account Dashboard (click this right above the Develop and Monitor tabs, and then scroll to the bottom of the For You tab).
        
    2.  AUTH_TOKEN -- replace this with your Auth Token, which can be found on your Account Dashboard (click this right above the Develop and Monitor tabs, and then scroll to the bottom of the For You tab).
        
    3.  FUNCTION_DOMAIN -- replace this with your Functions and Assets domain for the voicemail service, such as voicemail-XXXX.twil.io .  Find by going to Develop -> Functions and Assets -> Services -> voicemail.  This will be the domain/hostname located right above the blue Deploy All button at the bottom left.
        
    4.  SYNC_SID -- replace this with your Sync Default Service SID, which can be found by:
        
        1.  Scroll to the bottom of the left-side pane, click "Explore Products +", click Developer Tools or scroll down to Developer Tools, and click the pin next to Sync.
        2.  Now from the left-side pane, click on Sync and choose Services.
        3.  Now you should see a service called Default Service, which will list its SID next to it.
    5.  YOUR_PHONE_NUMBER -- replace this with your phone number, not your Twilio phone number.  Use format +14012224567.
        
        ```JSON
        {
          "description": "Telemarketer Deterrent",
          "states": [
            {
              "name": "Trigger",
              "type": "trigger",
              "transitions": [
                {
                  "event": "incomingMessage"
                },
                {
                  "next": "get_twilio_sync_send_to_voicemail",
                  "event": "incomingCall"
                },
                {
                  "event": "incomingConversationMessage"
                },
                {
                  "event": "incomingRequest"
                },
                {
                  "event": "incomingParent"
                }
              ],
              "properties": {
                "offset": {
                  "x": 300,
                  "y": 130
                }
              }
            },
            {
              "name": "gather_1",
              "type": "gather-input-on-call",
              "transitions": [
                {
                  "next": "split_1",
                  "event": "keypress"
                },
                {
                  "event": "speech"
                },
                {
                  "next": "http_1",
                  "event": "timeout"
                }
              ],
              "properties": {
                "voice": "Polly.Joanna",
                "number_of_digits": 1,
                "speech_timeout": "auto",
                "offset": {
                  "x": 440,
                  "y": 430
                },
                "loop": 1,
                "finish_on_key": "#",
                "say": "If this is urgent, press 1 now.  If you are a telemarketer, please place me on your do not call list and disconnect.  All other callers, please press 2 or hold on the line.",
                "language": "en-US",
                "stop_gather": true,
                "gather_language": "en",
                "profanity_filter": "true",
                "timeout": 5
              }
            },
            {
              "name": "split_1",
              "type": "split-based-on",
              "transitions": [
                {
                  "next": "set_twilio_sync_send_to_voicemail",
                  "event": "noMatch"
                },
                {
                  "next": "set_twilio_sync_send_to_voicemail",
                  "event": "match",
                  "conditions": [
                    {
                      "friendly_name": "If value equal_to 1",
                      "arguments": [
                        "{{widgets.gather_1.Digits}}"
                      ],
                      "type": "equal_to",
                      "value": "1"
                    }
                  ]
                },
                {
                  "next": "set_twilio_sync_send_to_voicemail",
                  "event": "match",
                  "conditions": [
                    {
                      "friendly_name": "If value equal_to 2",
                      "arguments": [
                        "{{widgets.gather_1.Digits}}"
                      ],
                      "type": "equal_to",
                      "value": "2"
                    }
                  ]
                }
              ],
              "properties": {
                "input": "{{widgets.gather_1.Digits}}",
                "offset": {
                  "x": 130,
                  "y": 670
                }
              }
            },
            {
              "name": "http_1",
              "type": "make-http-request",
              "transitions": [
                {
                  "next": "split_2",
                  "event": "success"
                },
                {
                  "next": "set_twilio_sync_send_to_voicemail",
                  "event": "failed"
                }
              ],
              "properties": {
                "offset": {
                  "x": 860,
                  "y": 420
                },
                "method": "GET",
                "content_type": "application/x-www-form-urlencoded;charset=utf-8",
                "add_twilio_auth": false,
                "parameters": [
                  {
                    "value": "icehook_scout",
                    "key": "AddOns"
                  }
                ],
                "url": "https://ACCOUNT_SID:AUTH_TOKEN@lookups.twilio.com/v1/PhoneNumbers/+{{trigger.call.From}}/"
              }
            },
            {
              "name": "split_2",
              "type": "split-based-on",
              "transitions": [
                {
                  "next": "set_twilio_sync_send_to_voicemail",
                  "event": "noMatch"
                },
                {
                  "next": "set_twilio_sync_send_to_voicemail",
                  "event": "match",
                  "conditions": [
                    {
                      "friendly_name": "If value less than 40",
                      "arguments": [
                        "{{widgets.http_1.parsed.add_ons.results.icehook_scout.result.risk_level}}"
                      ],
                      "type": "less_than",
                      "value": "40"
                    }
                  ]
                },
                {
                  "next": "say_play_1",
                  "event": "match",
                  "conditions": [
                    {
                      "friendly_name": "If value greater than 39",
                      "arguments": [
                        "{{widgets.http_1.parsed.add_ons.results.icehook_scout.result.risk_level}}"
                      ],
                      "type": "greater_than",
                      "value": "39"
                    }
                  ]
                }
              ],
              "properties": {
                "input": "{{widgets.http_1.parsed.add_ons.results.icehook_scout.result.risk_level}}",
                "offset": {
                  "x": 1230,
                  "y": 610
                }
              }
            },
            {
              "name": "say_play_1",
              "type": "say-play",
              "transitions": [
                {
                  "next": "record_voicemail_1",
                  "event": "audioComplete"
                }
              ],
              "properties": {
                "voice": "Polly.Joanna",
                "offset": {
                  "x": 1440,
                  "y": 960
                },
                "loop": 1,
                "say": "Please leave a message after the beep.",
                "language": "en-US"
              }
            },
            {
              "name": "record_voicemail_1",
              "type": "record-voicemail",
              "transitions": [
                {
                  "next": "say_play_2",
                  "event": "recordingComplete"
                },
                {
                  "event": "noAudio"
                },
                {
                  "event": "hangup"
                }
              ],
              "properties": {
                "transcribe": true,
                "offset": {
                  "x": 1190,
                  "y": 1220
                },
                "trim": "trim-silence",
                "transcription_callback_url": "https://ACCOUNT_SID:AUTH_TOKEN@FUNCTION_DOMAIN/t?f={{trigger.call.From}}",
                "play_beep": "true",
                "finish_on_key": "#",
                "timeout": 5,
                "max_length": 3600
              }
            },
            {
              "name": "say_play_2",
              "type": "say-play",
              "transitions": [
                {
                  "event": "audioComplete"
                }
              ],
              "properties": {
                "voice": "Polly.Joanna",
                "offset": {
                  "x": 1210,
                  "y": 1470
                },
                "loop": 1,
                "say": "Message received.  Good bye.",
                "language": "en-US"
              }
            },
            {
              "name": "set_twilio_sync_send_to_voicemail",
              "type": "make-http-request",
              "transitions": [
                {
                  "next": "connect_call_1",
                  "event": "success"
                },
                {
                  "next": "connect_call_1",
                  "event": "failed"
                }
              ],
              "properties": {
                "offset": {
                  "x": 730,
                  "y": 990
                },
                "method": "POST",
                "content_type": "application/x-www-form-urlencoded;charset=utf-8",
                "add_twilio_auth": false,
                "parameters": [
                  {
                    "value": "send_to_voicemail",
                    "key": "UniqueName"
                  },
                  {
                    "value": "{'send_to_voicemail': \"{{trigger.call.From}}\"}",
                    "key": "Data"
                  },
                  {
                    "value": "30",
                    "key": "Ttl"
                  }
                ],
                "url": "https://ACCOUNT_SID:AUTH_TOKEN@sync.twilio.com/v1/Services/SYNC_SID/Documents"
              }
            },
            {
              "name": "connect_call_1",
              "type": "connect-call-to",
              "transitions": [
                {
                  "event": "callCompleted"
                },
                {
                  "event": "hangup"
                }
              ],
              "properties": {
                "offset": {
                  "x": 670,
                  "y": 1260
                },
                "caller_id": "{{trigger.call.From}}",
                "noun": "number",
                "to": "YOUR_PHONE_NUMBER",
                "timeout": 15
              }
            },
            {
              "name": "get_twilio_sync_send_to_voicemail",
              "type": "make-http-request",
              "transitions": [
                {
                  "next": "split_send_to_voicemail",
                  "event": "success"
                },
                {
                  "next": "gather_1",
                  "event": "failed"
                }
              ],
              "properties": {
                "offset": {
                  "x": -330,
                  "y": 400
                },
                "method": "GET",
                "content_type": "application/x-www-form-urlencoded;charset=utf-8",
                "add_twilio_auth": false,
                "url": "https://ACCOUNT_SID:AUTH_TOKEN@sync.twilio.com/v1/Services/SYNC_SID/Documents/send_to_voicemail"
              }
            },
            {
              "name": "split_send_to_voicemail",
              "type": "split-based-on",
              "transitions": [
                {
                  "next": "gather_1",
                  "event": "noMatch"
                },
                {
                  "next": "say_play_1",
                  "event": "match",
                  "conditions": [
                    {
                      "friendly_name": "If value equal_to {{trigger.call.From}}",
                      "arguments": [
                        "{{widgets.get_twilio_sync_send_to_voicemail.parsed.data.send_to_voicemail}}"
                      ],
                      "type": "equal_to",
                      "value": "{{trigger.call.From}}"
                    }
                  ]
                }
              ],
              "properties": {
                "input": "{{widgets.get_twilio_sync_send_to_voicemail.parsed.data.send_to_voicemail}}",
                "offset": {
                  "x": -370,
                  "y": 670
                }
              }
            }
          ],
          "initial_state": "Trigger",
          "flags": {
            "allow_concurrent_calls": true
          }
        }
        ```
        

### Setting up Twilio: Phone Number

1.  Under the Develop tab, click on Phone Numbers, choose Manage, and choose Active Numbers.  Find and click on the Twilio Phone Number you wish to use with this Studio Flow.
2.  Under the Configure tab, scroll down to the **Voice Configuration** and set the following options accordingly:
    1.  Configure With: Webhook, TwiML, Bin, Function, Studio Flow, Proxy Service
    2.  A Call Comes In: Studio Flow
    3.  Flow: Telemarketer Deterrent
3.  Under the Configure tab, scroll down to the **Messaging Configuration** and set the following options accordingly:
    1.  Messaging Service: &lt;Be sure to choose the Messaging Service you configured earlier as part of the Twilio setup guide linked in the "Setting up Twilio" section above.&gt;
4.  Click Save Configuration at the bottom.

&nbsp;

### Finally: Forward calls to your Twilio Phone Number

How does this work?

1.  After setting up call forwarding (described below), any callers not in your contact list will be blocked by RoboBlocker, which effectively will call forward to your Twilio Phone Number.  You may get a RoboBlocker notification in your notification bar, but that's it, and you can turn that off, too (press and hold the app icon, go into App Info, and change the notifications accordingly).  No ring, no popup, no interruption.  Great!
2.  If the Twilio Studio Flow calls back to your phone number from the Twilio Studio Flow (either by pressing a number or holding on the line and having an IceHook Scout spam score < 40%), RoboBlocker will allow it to ring (the second call from the same caller phone number within 10 minutes).
3.  You may choose to decline the call or let it ring and miss the call, at which point the call will be forwarded back to your Twilio Phone Number and sent to your Twilio voicemail.  If the caller leaves a voicemail, you should receive a text message with the voicemail transcription and a link to play the voicemail audio.

How do I setup the call forwarding?

1.  Open your phone app, and dial: **\*\*004\*&lt;twilio phone number, ie 8885551234&gt;#**
2.  If you need to clear the call forwarding, open your phone app and dial: **##004#**
3.  If you get an error when clearing or setting your call forwarding, I've found that you have only so many times you can set/clear it before you have to reset your mobile network settings.
    1.  For Android 15 (on Google Pixel 9 series): Go into your Android Settings, tap on System, tap on Reset Options, and tap on Reset Mobile Network Settings, and this should get you back to where you can successfully dial the above call forwarding codes.  My phone provider helped me with this and this worked for me.

&nbsp;

### Testing your setup

I highly recommend testing your setup.  Here are some ideas.

- Call your phone number from another phone, someone from your contact list is perfect.
- Call your phone number from another phone, someone from your contact list is perfect: this time, slightly change their phone number in your contacts, so that their real number is no longer in your contacts.
- Call your phone number from Google Voice on your phone (get a Google Voice number within the Google Voice app, and ensure you've enabled WiFi calling within the Google Voice app, so that you actually call your real number from your Google Voice number).
- You might make a new Studio Flow that says the IceHook Scout spam score for the number you're calling from.  When I was testing with my Google Voice number, the spam score was at or just above 40%, meaning I couldn't test part of the regular Studio Flow unless I adjusted the spam score checks in the regular Studio Flow.  Just be sure to set them back or adjust them accordingly.

&nbsp;

### Disclaimer Statement and License

**You should be comfortable with setting something like this up.  You should understand what you're doing and how this may affect your daily calls, including any emergency calls, etc. that you would not normally expect.  You hereby accept and acknowledge that you take full responsibility for your use of this documentation and that I (Matt Smith) will not and cannot be held liable for your use of this documentation, in whole or in part, for any reason, whatsoever.**

This work is licensed under the MIT License.

&nbsp;

### Final Words

I wish I could export my Twilio account's configuration, allowing you to easily import the whole thing.  Unfortunately, that's not possible, and made the documentation longer than I would have liked, though it is a somewhat complex (really complex?) setup.  Like I said, it's overkill.  Anyways, I hope you've found this useful, either in its entirety or in part.
