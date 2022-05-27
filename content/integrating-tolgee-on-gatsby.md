Tolgee is an open-source translation management system, which is designed to make the process of localization faster and easier. 

The tolgee architecture would consist of a Server which is a REST-API backend as a service, the web application and integration libraries. You store your localized strings in the backend, and the integration libraries can use this data via REST API in your web app.

Gatsby is an open-source framework that combines functionality from React, GraphQL and Webpack into a single tool for building static websites and apps



To get started with a sample gatsby project, install the `@tolgee/react` module

`npm install @tolgee/react`

Sign in/up on [tolgee platform](https://app.tolgee.io/) and create a project, with the languages you want to translate. In my case I want to translate from English to Swahili

![](/home/jimii/Documents/webcode/jimii/static/tolgee-gatsby-project/Screenshot_20220527191027.png)  



We will add a few translations for both languages. This we will do by going to the 'Translation' tab on the left pane of the Tolgee dashboard. In our case our key name will be `greetings`

Next generate the API keys by going to the integrate pane on the left side bar, chose the development weapon of choice, in our case Gatsby and finally select the scope of the API key. After this part, the generated key comes with very nice documentation on how to use tolgee with your Gatsby project.



If the `.env.development` file does not exist, create it and add the generated api key along the tolgee url

```shell
GATSBY_TOLGEE_API_KEY=<API KEY>
GATSBY_TOLGEE_API_URL=https://app.tolgee.io
```

Create an `src/i18n` folder if it doesn't exist. Export your translations as JSON and save them to this `i18n` folder .

Install the  `gatsby-plugin-intl` using npm. This plugin uses [react-intl](https://github.com/yahoo/react-intl) to internationalize your site. On production Gatsby will generate pages separately for each locale, so only locale that is needed is provided statically.

`npm install -D gatsby-plugin-react-intl` 

Add the plugin to your `gatsby-config.js`

```js
module.exports = {
  plugins: [
    {
      resolve: `gatsby-plugin-react-intl`,
      options: {
        path: `${__dirname}/src/i18n`,
        languages: [`en`, `cs`],
        defaultLanguage: `en`,
        redirect: true,
      },
    },
  ],
};
```

Now we make use of Tolgee

```js
import { useIntl } from 'gatsby-plugin-react-intl';
import { TolgeeProvider } from '@tolgee/react';

import * as translationsEn from '../i18n/en.json';
import * as translationsCs from '../i18n/sw.json';

export const AppWrapper: React.FC = ({ children }) => {
  const intl = useIntl();

  return (
    <TolgeeProvider
      forceLanguage={intl.locale}
      apiKey={process.env.GATSBY_TOLGEE_API_KEY}
      apiUrl={process.env.GATSBY_TOLGEE_API_URL}
      staticData={{
        en: translationsEn,
        cs: translationssw,
      }}
      loadingFallback={<div>Loading...</div>}
    >
      {children}
    </TolgeeProvider>
  );
};
```

Now in the desired component/page to translate, you can use Tolgee

```js
import {T} from "@tolgee/react";

...

<T>translation_key</T>
```

or 

```js

```


