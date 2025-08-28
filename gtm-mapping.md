GTM Mapping Guide

This guide maps Data Layer values from Craft CMS to Google Tag Manager variables and GA4 event parameters. It also covers triggers, naming, consent and QA. Use NZD as the default currency unless your site supports multi-currency.

Naming conventions

DLV variables in GTM: DLV - <path>
Example: DLV - transaction_id, DLV - items, DLV - userData.user_id

GA4 Config tag: GA4 - Config

GA4 Event tags: GA4 - <event name>

Custom event triggers: EV - <event name>

Core GA4 setup in GTM

GA4 Config tag

Tag: Google Analytics: GA4 Configuration

Measurement ID: {{GA4 - ID}}

Trigger: All pages

Send page view: off if you send page_view separately, on if you prefer automatic PVs

Consent: Enabled. Respect region rules.

Data Layer Variables to create

Create these DLVs in GTM. Set Version to Version 2. Leave default values blank unless noted.

GTM Variable name	Data Layer Variable name	Notes
DLV - currency	currency	String
DLV - transaction_id	transaction_id	String
DLV - value	value	Number
DLV - tax	tax	Number
DLV - shipping	shipping	Number
DLV - coupon	coupon	String or null
DLV - payment_type	payment_type	String
DLV - items	items	Array of objects
DLV - method	method	For sign_up or newsletter
DLV - plan	plan	Wine club plan handle
DLV - user_id	user_id	Non-PII numeric ID only
DLV - form_id	form_id	Newsletter form identifier
DLV - source_page	source_page	Path or URI
DLV - channel_group	channel_group	Social, referral, etc
DLV - source	source	Traffic source
DLV - medium	medium	Medium
DLV - campaign	campaign	Campaign

If you also push nested objects, create variables like:

DLV - userData.user_id with Data Layer Variable Name userData.user_id

Event to parameter mapping

Create one GA4 Event tag per event. Use a Custom Event trigger that matches the event value in the push.

1) view_item

Trigger: EV - view_item
Custom Event equals view_item

Tag: GA4 - view_item

Parameters:

currency → {{DLV - currency}}

value → {{DLV - value}}

items → {{DLV - items}}

2) add_to_cart

Trigger: EV - add_to_cart

Tag: GA4 - add_to_cart

Parameters:

currency → {{DLV - currency}}

value → {{DLV - value}}

items → {{DLV - items}}

3) view_cart

Trigger: EV - view_cart

Tag: GA4 - view_cart

Parameters:

currency → {{DLV - currency}}

value → {{DLV - value}}

items → {{DLV - items}}

4) begin_checkout

Trigger: EV - begin_checkout

Tag: GA4 - begin_checkout

Parameters:

currency → {{DLV - currency}}

value → {{DLV - value}}

items → {{DLV - items}}

5) add_shipping_info (optional, at shipping step)

Trigger: EV - add_shipping_info

Tag: GA4 - add_shipping_info

Parameters:

currency → {{DLV - currency}}

value → {{DLV - value}}

shipping_tier → {{DLV - shipping_tier}} Create DLV if you push it

items → {{DLV - items}}

6) add_payment_info (optional, at payment step)

Trigger: EV - add_payment_info

Tag: GA4 - add_payment_info

Parameters:

currency → {{DLV - currency}}

value → {{DLV - value}}

payment_type → {{DLV - payment_type}}

items → {{DLV - items}}

7) purchase

Trigger: EV - purchase

Tag: GA4 - purchase

Parameters:

currency → {{DLV - currency}}

transaction_id → {{DLV - transaction_id}}

value → {{DLV - value}}

coupon → {{DLV - coupon}}

payment_type → {{DLV - payment_type}}

tax → {{DLV - tax}}

shipping → {{DLV - shipping}}

items → {{DLV - items}}

De-dupe: keep the page script guard that stores a dl_purchase_<transaction_id> flag in sessionStorage. GA4 also uses transaction_id for internal de-duplication across web and app. Using both is safest.

8) sign_up (wine club)

Trigger: EV - sign_up

Tag: GA4 - sign_up

Parameters:

method → {{DLV - method}} Example: wine_club

plan → {{DLV - plan}} Custom dimension if you want to analyse plan types

user_id → {{DLV - user_id}} Only non-PII. Consider mapping this to GA4 User ID via the GA4 Config tag if you run a signed-in experience

9) newsletter_subscription

Trigger: EV - newsletter_subscription

Tag: GA4 - newsletter_subscription

Parameters:

method → {{DLV - method}} Example: form, popup, checkout

form_id → {{DLV - form_id}}

source_page → {{DLV - source_page}}

10) traffic_source (social start helper)

Trigger: EV - traffic_source

Tag: GA4 - traffic_source

Parameters:

channel_group → {{DLV - channel_group}} Example: social

source → {{DLV - source}}

medium → {{DLV - medium}}

campaign → {{DLV - campaign}}

This event is a helper for reporting. GA4 already has session_start. Do not duplicate that. Use this only to enrich attribution for social if you want a clean event to pivot on.

Recommended GA4 custom dimensions and metrics

Create these in GA4 Admin → Custom definitions.

Scope	Name	Event parameter
Event	plan	plan
Event	method	method
Event	payment_type	payment_type
Event	shipping_tier	shipping_tier
Event	form_id	form_id
Event	source_page	source_page
Event	channel_group	channel_group
Event	traffic_source	source
Event	traffic_medium	medium
Event	traffic_campaign	campaign

User-scoped ID: if you run a signed-in experience and you are confident about policy compliance, set User-ID in the GA4 Config tag using {{DLV - user_id}}. Never send emails or names to GA4.

Triggers to create

Create a Custom Event trigger per event:

Trigger name	Type	Event name
EV - view_item	Custom Event	view_item
EV - add_to_cart	Custom Event	add_to_cart
EV - view_cart	Custom Event	view_cart
EV - begin_checkout	Custom Event	begin_checkout
EV - add_shipping_info	Custom Event	add_shipping_info
EV - add_payment_info	Custom Event	add_payment_info
EV - purchase	Custom Event	purchase
EV - sign_up	Custom Event	sign_up
EV - newsletter_subscription	Custom Event	newsletter_subscription
EV - traffic_source	Custom Event	traffic_source

Add trigger exceptions for admin, staging and preview paths if you prefer not to send events there.

Consent and compliance

Enable Consent Mode in GTM.

Ensure GA4 event tags only fire when consent is granted for the relevant purposes.

Do not send PII to GA4. No emails, names, phone numbers or addresses.

Admin and staging exclusions

Create a Trigger Group or Exceptions such as:

Page URL contains /admin

Hostname matches staging.poppiesmartinborough.co.nz or your staging domain

Environment variable flag if you set one in Craft

Debugging checklist

Open GTM Preview and load a page.

Confirm dataLayer contains the expected push with the right event.

Check your Custom Event trigger fires.

Open the GA4 Event tag and confirm parameters are populated with DLVs.

In GA4 DebugView, verify the event arrives with items and values.

For purchase, refresh the confirmation page. Confirm the de-dupe guard prevents a second push.

Default values and fallbacks

Currency default: NZD

If a field can be null, let it be null rather than sending empty strings.

Values should be numbers, not strings. Format in Twig with number_format(2, '.', '') where relevant.
