# test
exports.handler = async function handler(context, event, callback) {
  try {
    const runtime = require('twilio').default.runtime;
    let twiml = new Twilio.twiml.VoiceResponse();

    const openingHoursFunctionURL = 'is-regular-weekday-opening-hours';
    const specialDayFunctionURL = 'is-special-day';

    const openingHoursFunction = runtime.services(context.SERVICE_SID).functions.function(openingHoursFunctionURL);
    const responseOpeningHours = await openingHoursFunction.invoke();
    const isRegularWeekdaysOpeningHours = responseOpeningHours.data;

    const specialDayFunction = runtime.services(context.SERVICE_SID).functions.function(specialDayFunctionURL);
    const responseSpecialDay = await specialDayFunction.invoke();
    const isSpecialDay = responseSpecialDay.data;

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
