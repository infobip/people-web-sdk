# People Web SDK

The People Web SDK is designed and developed so you can easily integrate your website with `People`.  Here you will find an overview, a quick guide on how to connect to Infobip platform via Web SDK, track visitor behavior, personalize and update profiles and also see the functionality in action with some examples to get you started.

Use Web SDK for progressive customer profiling by:

- Tracking activity of both anonymous and known visitors on your website (sessions, pages viewed, custom site actions that you define).
- Converting anonymous visitors to new Leads or matching to existing Lead profiles as they fill out a web form or subscribe to a newsletter and merging their entire pre-identification website activity to the identified profile.
- Matching anonymous visitors to existing Customer profiles as they log into their site account and merging their pre-login session events into the Customer profile timeline. 
- Updating Lead profiles with information collected from identified users on the website.

To get started with the Web SDK take a look at the methods and descriptions below. 

|METHOD NAME|DESCRIPTION|
|-------------|-------------|
|init|Add init to every page on your website that you want to track activity|
|setPerson|Connects the current user session to an known profile in People|
|updatePerson|Update profile attributes with data from the website|
|track|Sends a custom event to a visitor, known lead or customer|
|appendToList|Adds a new item to a List profile attribute|
|forgetPerson|Reverts setPerson by resetting  the user context and closing the current session. Typically used when a user logs off|

## Browser Compatibility

We support up to 2 most recent versions of these browsers (unless otherwise indicated):

||Google Chrome|Mozilla Firefox|Microsoft Edge|Safari|Opera|
|-------------|-------------|-------------|-------------|-------------|-------------|
|Minimum Browser Version|v.94 - v.96|v.94 - v.95|v.95 - v.96|v.14 - v.15|	v.81 - v.82|
  
## Deploy the Library
  
To start tracking with the Javascript library you will need to add the library which includes the `init` method inside the `head` tags on all pages of the website that you want to track. 

1. Copy the following code snippet to your website.  If you already have an Infobip [account](https://www.infobip.com/docs/essentials/create-account), you will find the code snippet pre-populated with an API key for your account in the [Web SDK](https://portal.infobip.com/people/events/websdk) section of the portal.
2. Generate a new API key on the [Manage API Keys](https://portal.infobip.com/settings/accounts/api-keys) page of the portal. Remember to enable the Web SDK role for the key.
3. Add the new custom API key to your code snippet.

```javascript
<script>
(function(e,t,n,o){e.PeopleEventsObject=o;e[o]=e[o]||{init:function(t){e[o].apiKey=t},
setPerson:function(t,n){e[o].person=t;e[o].personTtl=n},forgetPerson:function(){e[o].toForgetPerson=true},track:function(){(e[o].q=e[o].q||[]).push(arguments)},
updatePerson:function(t){e[o].personToUpdate={person:t}},appendToList:function(t,n){e[o].attributeToAppend={attributeName:t,attribute:n}}};var r=t.createElement("script");
var s=t.getElementsByTagName("script")[0];r.async=1;r.src=n;s.parentNode.insertBefore(r,s)})
(window,document,"https://s3.eu-central-1.amazonaws.com/portal-cdn-production/people-events-sdk/pe.latest-2.js","pe");
 
pe.init('Your API Key');
</script>
```

## Track Visitor Behavior
Whenever a site visitor opens a web page, the Web SDK is loaded and starts tracking the visitor's activity on the page.

The tracking comprises of four steps:

1. Establish user context

The Web SDK checks if the visitor is a new or returning anonymous visitor or a known person whose identity was established on a previous page. For a new anonymous user, the visitor is assigned a tracking ID (visitor ID). In case the user is a returning anonymous or known visitor, the Web SDK uses their existing tracking ID (visitor ID or person ID) and resets the tracking expiration timer. Take a look at the Visitor Personalization section below to learn more about setting the user context in the Web SDK.

**NOTE**
A 45 day idle period for anonymous visitors is applied to the Web SDK by default. This means that the system will remove all visitor data (session activity and visitorID) after 45 days of inactivity.  If the user returns to the site after 45 days then they will be treated as a new anonymous visitor in the system and assigned a new visitorID. If the visitor returns to the website within 45 days then they will be treated as a known visitor and their page view and session activity will be tracked under the same visitorID.

2. Track user session

When the visitor enters the website through the page, the Web SDK creates a new user session to keep all site events the user triggers during his visit. The SDK sends the standard Session Started event when a session is created. If the page is not the first one that the visitor visits on the website, the SDK keeps the existing user session running. The session expires after 30 minutes of inactivity (no events registered) or the website explicitly closes the session by calling `pe.forgetPerson()`. Once the session is closed, the SDK sends the standard Session Ended event. For more information about Session Started/Ended events see [People Events](https://www.infobip.com/docs/people/events#people-events-standard-events).

**NOTE**
You will need to enable tracking of the Session Started and Session Ended events in People Events configuration.

3. Log Page View event

The Web SDK automatically sends the standard Page View event to the visitor's timeline at the page load. See [People Events](https://www.infobip.com/docs/people/events#people-events-standard-events) for more information about what visitor and page properties the Page View event captures.

**NOTE**
You will need to enable tracking of the Page View event in the People Events configuration.

4. Track custom events

You can use the `pe.track()` SDK method to track your custom events based on the visitor's activity on the page.

You must pass the event Definition ID (which can be found on the event definitions page of the portal) and event properties to the track method. See [Track Custom Events](https://www.infobip.com/docs/people/events#track-custom-events) to learn more.

```javascript
pe.track('definitionId', { propertyName: 'propertyValue' });
```
Always check to make sure that your properties match your definition. The properties may contain either all fields from the definition or some of them, but they can't contain anything that wasn't defined as a property. 

For further information take a look at tracking [custom events](https://www.infobip.com/docs/people/events#track-custom-events) documentation here.

## Visitor Personalization
Once the visitor is identified on the website (e.g., when the visitor provides their email or phone number in a web form, or by registering on the site), you can call the `pe.setPerson()` method to map the visitor to either new or existing People profile.

You need to pass the collected contact information in your call, which will result in either outcome:

1. New Lead profile created in People when there is no matching profile with the specified email or phone number.

2.  Existing Lead or Customer profile is set as user context in the Web SDK  when the contact information in the profile matches the supplied email, phone number, external customer identifier or push registration identifier. 

```javascript
// set person using email
pe.setPerson({ email: 'someone@email.com'}, 2000 );
// where 2000 is expiration period in seconds.
// By default, expiration period is 1800 seconds (30 minutes).
// alternatively you can set person using phone
// pe.setPerson({ phone: '4411123456' });
// or by push registration identifier
// pe.setPerson({ pushRegId: '123456' });
```

**NOTE** 
You can set the optional personalization expiration period in your call. If no value is passed in the method call then the expiration time is set to 1800 seconds (30 minutes). The connection to a known People profile established by `pe.setPerson()` expires when there is no activity registered in the personalized user session. If the user becomes active again or returns to the web site, the SDK will track them as a new anonymous visitor.  

When you call `pe.setPerson()` on an identified site visitor, the visitor's browsing and event history is merged with the matched profile depending on the profile type:

For Lead profiles, all recorded user sessions are appended to the profile to provide richer context for lead nurturing.
For Customer profiles only the current session events are appended to the profile.
Once the `setPerson` call established the connection to a known People profile and the browsing and event history is merged from the anonymous visitor, the anonymous tracking data is deleted and the visitor tracking continues using the profile ID (person ID) .

You may also choose to reset the user context when the user logs off from his account. To do that invoke `pe.forgetPerson()` to clear the personalization information, end the user session and continue tracking the user as a new anonymous visitor.   
```javascript
// forget person
pe.forgetPerson();
```

**NOTE**
Calling `pe.forgetPerson()` on an anonymous visitor will simply end the current user session. The first event the visitor triggers will start a new session.  

## Update Profile Information
You can use the Web SDK whenever there's a need to align the profile changes happening on your website with the People platform. This can be done using the `updatePerson` method and `appendToList` method (if you need to append a new object to a list type custom attribute). 

**IMPORTANT**
Only Lead profiles can be updated by Web SDK. Updating Customers over Web SDK is restricted.

You will need to copy the below code to your website and adapt it to include the custom attribute(s) you wish to update in People.

Take a look at the [person profile API documentation](https://www.infobip.com/docs/api#customer-engagement/people) for more information about the object details and how the custom attribute should be formatted.

```javascript
// update person
pe.updatePerson({ personAttribute: 'personAttributeValue' });
```
If the update requires you need to append a new object to a list type custom attribute then you can use the `appendToList` method.

The [person profile API documentation](https://www.infobip.com/docs/api#customer-engagement/people) shows the format you will need to apply for list type attributes so you can adapt the code to your needs.

```javascript
// appends attribute to list
pe.appendToList('attributeName', { personAttribute: 'personAttributeValue' });
```

## Web SDK Usage Examples
We have included a few examples of the Web SDK calls in context to help you with your integrations.

**NOTE**  
The methods of the Web SDK return a special JavaScript object called a promise. We recommend that you properly handle the returned promises using either `async/await` or `then()` JavaScript constructs to ensure a proper execution order. The code samples in this chapter show how you can do it.

### User Login
Using Async / Await

```javascript
const onLoginFormSubmit = async (evt) => {
    evt.preventDefault();
    /* ... */
    await pe.setPerson({ email: 'someone@email.com' }); // profile identifier from the login form
    await pe.track('login'); // your custom event
};
 
const loginForm = document.querySelector('form');
loginForm.addEventListener('submit', onLoginFormSubmit);
```

Using Then()

```javascript
const onLoginFormSubmit = (evt) => {
  evt.preventDefault();
  const formData = new FormData(document.querySelector('form'));
  const formEmail = formData.get('email');

  pe.setPerson({ email: formEmail }).then(() => {
    return pe.track('login'); // your custom event
  });
};

const loginForm = document.querySelector('form');
loginForm.addEventListener('submit', onLoginFormSubmit);
```

### User Signup
Using Async / Await

```javascript
const onRegistrationFormSubmit = async (evt) => {
    evt.preventDefault();
    /* ... */
    await pe.setPerson({ email: 'someone@email.com' }); // profile identifier from the registration form
    await pe.track('registration'); // your custom event
    await pe.updatePerson({ firstName: 'First name', lastName: 'Last name' }); // person attributes from the form
};
 
const registrationForm = document.querySelector('form');
registrationForm.addEventListener('submit', onRegistrationFormSubmit);
```

Using Then()

```javascript
const onRegistrationFormSubmit = (evt) => {
  evt.preventDefault();
  const formData = new FormData(document.querySelector('form'));
  const formEmail = formData.get('email');
  const formFirstName = formData.get('first-name');
  const formLastName = formData.get('last-name');

  pe.setPerson({ email: formEmail })
    .then(() => pe.track('registration')) // your custom event
    .then(() => pe.updatePerson({ firstName: formFirstName, lastName: formLastName }));
};

const registrationForm = document.querySelector('form');
registrationForm.addEventListener('submit', onRegistrationFormSubmit);
```

### User Logoff

```javascript
const onLogoutButtonClick = async () => {
    await pe.track('logout'); // your custom event
    await pe.forgetPerson();
};
 
const logoutButton = document.querySelector('.logout-button');
logoutButton.addEventListener('click', onLogoutButtonClick);
```

## Ask For Help
Feel free to open issues on the repository for any issue or feature request. 

If it is, however, something that requires our imminent attention feel free to contact us @ support@infobip.com.
