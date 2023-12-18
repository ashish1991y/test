exports.handler = async function handler(context, event, callback) {
  try {
    const runtime = require('twilio').default.runtime;
    let twiml = new Twilio.twiml.VoiceResponse();

    const openingHoursFunctionURL = 'is-regular-weekday-opening-hours';
    const specialDayFunctionURL = 'is-special-day';

    const openingHoursFunction = runtime.services(context.SERVICE_SID).functions.function(openingHoursFunctionURL);
    const specialDayFunction = runtime.services(context.SERVICE_SID).functions.function(specialDayFunctionURL);

    // Use context.ACCOUNT_SID and context.AUTH_TOKEN directly in the authentication header
    const auth = 'Basic ' + Buffer.from(context.ACCOUNT_SID + ':' + context.AUTH_TOKEN).toString('base64');

    const headers = {
      'Authorization': auth,
      'Content-Type': 'application/json', // Adjust based on your function's expected content type
    };

    const options = {
      method: 'POST',
      headers: headers,
    };

    // Invoke openingHoursFunction
    const responseOpeningHours = await openingHoursFunction.invoke(options);
    const isRegularWeekdaysOpeningHours = responseOpeningHours.data;

    // Invoke specialDayFunction
    const responseSpecialDay = await specialDayFunction.invoke(options);
    const isSpecialDay = responseSpecialDay.data;

    // Your logic based on function responses
    if (isRegularWeekdaysOpeningHours || isSpecialDay) {
      switch (event.Digits) {
        case '1':
          twiml.redirect({ method: 'POST' }, 'https://calamity-menu-particulieren-4256.twil.io/CalamityMenuParticulieren');
          break;
        case '2':
          twiml.redirect({ method: 'POST' }, 'https://zapping-zakelijk-2705.twil.io/Zappingzakelijk');
          break;
        default:
          twiml.play({ loop: 1 }, 'https://disillusioned-jellyfish-8740.twil.io/assets/NL-NL-ALG-Welkom-1-wav');
          twiml.gather({ numDigits: 1, timeout: 6 })
            .play({ loop: 1 }, 'https://disillusioned-jellyfish-8740.twil.io/assets/C2657-1P-2Z.wav');
      }
    } else {
      const redirectURI = isSpecialDay ? 'https://disillusioned-jellyfish-8740.twil.io/assets/Generic_closed820holiday.wav' : 'https://disillusioned-jellyfish-8740.twil.io/assets/Generic';
      twiml.play({ loop: 1 }, redirectURI);
    }

    callback(null, twiml);
  } catch (error) {
    console.error('Error:', error);
    callback('Application error', null);
  }
};
