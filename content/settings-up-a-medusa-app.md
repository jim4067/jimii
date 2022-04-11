# HOW TO SETUP A MEDUSA APP WITHOUT CREATE-MEDUSA-APP

**Keyword:** [Enter Targeted Keyword]

**Keyword MSV:** [Enter Targeted Keyword’s Monthly Search Volume]

**Author:** James Mutuku

**Due Date:** 11/04/2022

**Publish Date:** [Enter Desired Publish Date]

**Buyer Persona:** [Enter Targeted Reader and/or Buyer Persona]

## intro

With the onset of the Coronavirus disease, many business were forced to close down to inhibit or atleast try to minimize human contact as much as possible. With no clear timelines when these restrictions would be eased and businesses allowed to reopen, most of them shifted to marketing and selling their products and goods online. Online accounts selling goods, began showing up on social media platforms -Instragram, whatsapp, Facebook - you name it. But managing customer relations or dealing with inventory on social media platforms was painful and damn near impossible. You could hire a freelance developer to build an e-commerce site for your business and in doing so be prepared to put your kidneys for sale to finance your decision. With many organizations not ready to pay the gargantuan prices that freelance devs charge, many were forced to use e-commerce frameworks such as shopify to create their online stores. This would get their businesses on the internet but in using an e-commerce framework like shopify be ready to give up on customization and still pay for it.

With the pandemic under control in most parts of the world and economies opened up you priorities might not lie in opening up a online store. While the choice to not do so is valid there are so many advantaged that will come with having on online platform where you can advertise and show-off some of the spectacular unique goods you sale for potential new customers and give them the ability to give feedback on how you could better improve what you sell. 

## in this article

There are many ways to get your business online but in this tutorial we will be using an e-commerce framework. I did mention Shopify previously but due the rigid nature and inflexible design we will be using Medusa, an open source alternative to Shopify.

It is highly customizable, free and with thousand of community created plugins to extend its functionality you can be sure it won't get in the way of you building you e-commerce site. 

## how to

To get started with a [new medusa](https://docs.medusajs.com/quickstart/quick-start/) project install the medusa-cli client globally using [npm](https://docs.npmjs.com/cli/v8). On some Linux distros such as Debian 10, [node]([Node.js](https://nodejs.org/en/)) isn't installed but a quick `sudo apt install nodejs` should do the trick. But because of the archaic software that Debian ships with, I would highly recommending installing node and npm using [nvm]([GitHub - nvm-sh/nvm: Node Version Manager - POSIX-compliant bash script to manage multiple active node.js versions](https://github.com/nvm-sh/nvm))  

```bash
npm install -g @medusajs/medusa-cli
```

To make sure that medusa is installed correctly do a quick 

```bash
medusa --version
```

Which should produce output shown below for first time users 

![Medusa Cli Output](/static/setting-up-medussa/medusa-cli-first.jpg)

Now that we have medusa installed it's time to create our project. To do so we use

```bash
medusa new my-awesome-store --seed
```

After a couple of minutes while the medusa-cli client does its thing, the process should complete and you can now go into your `my-awesome-store` 

You can now start your development server using `medusa develop`. You now have a fully functioning e-commerce backend server. By default Medusa with user port 9000 and from here you should be able to access routes that serve the product details. For example you should have some dummy data because we used the `--seed` option if you navigate to http://localhost:9000/store/products on your browser.

![Seeded Medusa Data](/static/setting-up-medussa/sampleMedusaData.png)



On the front end you have two options that you can use to interact with your backend server. `nextjs` or `gatsbyjs` depending on which you prefer. I prefer using nextjs so I will head over to their [starter template](https://github.com/medusajs/nextjs-starter-medusa)

```bash
git clone https://github.com/medusajs/nextjs-starter-medusa.git

cd nextjs-starter-medusa
```

Inside there frontend template that you just cloned, you will need to rename the `.env.template` file to `.env.local` and update it with your Private Stripe api key `NEXT_PUBLIC_STRIPE_KEY=pk_test_something`. This nextjs starter uses yarn, so you'll need to install and start the dev server using 

```bash
yarn install
yarn dev 
```

The last part will require you to install the admin panel. This admin is crucial if your site will be managed by non-technical folks. The admin panel enables you to set up the stores configuration, manage simple things like the website's name, currencies or shipping methods. 



To get started with setting up the admin panel 

**clone the admin panel repo**

```bash
git clone https://github.com/medusajs/admin medusa-admin
cd medusa-admin
```

**install the dependencies**

```bash
yarn install
```

**start the development server**

```bash
yarn develop
```

This will start up the panel at part 7000, so head over to https://localhost:7000 and you should see a portal similar to the one below

![Medusa Admin Portal Login](/static/setting-up-medussa/medusaAdminPortal.jpg)

We seeded dummy data from medusa so to access the portal we can use the provided test credentials `admin@medusa-test.com` for our email and `supersecret` as our password. After a successful login the default screen as shown below will show up

![Medusa Adming Landing](/static/setting-up-medussa/medusaAdminLanding.jpg)

But the provided credentials are for testing whether we set up the portal correctly. To create our own user we will need to run the command below and set our email using and the password we want to use. Make sure to use Alphanumericals to make it more secure.

```bash
medusa user -e some@email.com -p some-password
```

Medusa is composed of three components, the headless backend, the admin dashboard, and the storefront i.e the frontend. The medusa docs are well documented and they are worth checking out for more fine-grained about building your platform.



And there you have. In under twenty commands you have fully set up medusa with the the backend, the frontend and the admin portal and you are now ready to begin customizing it as per your needs.

Medusa is one of the projects that are being onboarded at Aviyel. Check out their page to find out more [Aviyel | Medusa](https://aviyel.com/projects/10/medusa)



Join the [Aviyel]([Aviyel](https://discord.com/invite/sdxNdbANeX))
