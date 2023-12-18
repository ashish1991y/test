exports.handler = async function handler(context, event, callback) {
  try {
    const runtime = require('twilio').default.runtime;
    let twiml = new Twilio.twiml.VoiceResponse();

    const openingHoursFunctionURL = 'https://data-3232.twil.io/is-regular-weekday-opening-hours';
    const specialDayFunctionURL = 'https://data-3232.twil.io/is-special-day';

    const options = {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
    };

    // Invoke openingHoursFunction
    const responseOpeningHours = await runtime.request({ ...options, uri: openingHoursFunctionURL });
    console.log('Response from openingHoursFunction:', responseOpeningHours); // Add this line for logging
    const isRegularWeekdaysOpeningHours = responseOpeningHours.data;

    // Invoke specialDayFunction
    const responseSpecialDay = await runtime.request({ ...options, uri: specialDayFunctionURL });
    console.log('Response from specialDayFunction:', responseSpecialDay); // Add this line for logging
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
    callback('Application error: ' + error.message, null);
  }
};
