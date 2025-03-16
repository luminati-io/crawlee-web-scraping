# Web Scraping with Crawlee

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/) 

Learn how to use Crawlee for efficient [web scraping with Node.js](https://brightdata.com/blog/how-tos/web-scraping-with-node-js):

- [Basic Web Scraping with Crawlee](#basic-web-scraping-with-crawlee)
- [Proxy Rotation with Crawlee](#proxy-rotation-with-crawlee)
- [Sessions Management with Crawlee](#sessions-management-with-crawlee)
- [Dynamic Content Handling with Crawlee](#dynamic-content-handling-with-crawlee)

## Prerequisites

Before you start, make sure you have the following prerequisites installed:

* **[Node.js](https://nodejs.org/).**
* **[npm](https://www.npmjs.com/):** This typically comes with Node.js. You can verify the installation by running `node -v` or `npm -v` in your terminal.
* **A code editor of your choice:** This tutorial uses [Visual Studio Code](https://code.visualstudio.com/).

## Basic Web Scraping with Crawlee

Let’s start by scraping the [Books to Scrape](https://books.toscrape.com/) website.

Open your terminal or shell and initialize a Node.js project:

```bash
mkdir crawlee-tutorial
cd crawlee-tutorial
npm init -y
```

Install the Crawlee library:

```bash
npm install crawlee
```

To scrape data effectively, inspect the target website’s HTML structure. Open the site in your browser, right-click anywhere on the page, and select **Inspect** or **Inspect Element** in **Developer Tools**.

![Inspect HTML element](https://github.com/luminati-io/crawlee-web-scraping/blob/main/images/Inspect-HTML-element-1024x540.png)

The **Elements** tab in **Developer Tools** displays the page’s HTML layout. In this example:  

- Each book is inside an `article` tag with the class `product_pod`.  
- The book title is in an `h3` tag, with the actual title stored in the `title` attribute of the nested `a` tag.  
- The book price is inside a `p` tag with the class `price_color`.  

![Inspect the HTML elements on the Books to Scrape website](https://github.com/luminati-io/crawlee-web-scraping/blob/main/images/Inspect-the-HTML-elements-on-the-Books-to-Scrape-website-1024x522.png)

Under the root directory of your project, create a file named `scrape.js` and add the following code:

```js
const { CheerioCrawler } = require('crawlee');

const crawler = new CheerioCrawler({
    async requestHandler({ request, $ }) {
        const books = [];
        $('article.product_pod').each((index, element) => {
            const title = $(element).find('h3 a').attr('title');
            const price = $(element).find('.price_color').text();
            books.push({ title, price });
        });
        console.log(books);
    },
});

crawler.run(['https://books.toscrape.com/']);
```

This code uses `CheerioCrawler` from `crawlee` to extract book titles and prices from `https://books.toscrape.com/`. It fetches the HTML, selects `<article class="product_pod">` elements using jQuery-like syntax, and logs the results to the console.

After adding the code to your `scrape.js` file, run it with the following command:

```bash
node scrape.js
```

An array of book titles and prices should print to your terminal:

```
…output omitted…
  {
    title: 'The Boys in the Boat: Nine Americans and Their Epic Quest for Gold at the 1936 Berlin Olympics',
    price: '£22.60'
  },
  { title: 'The Black Maria', price: '£52.15' },
  {
    title: 'Starving Hearts (Triangular Trade Trilogy, #1)',
    price: '£13.99'
  },
  { title: "Shakespeare's Sonnets", price: '£20.66' },
  { title: 'Set Me Free', price: '£17.46' },
  {
    title: "Scott Pilgrim's Precious Little Life (Scott Pilgrim #1)",
    price: '£52.29'
  },
…output omitted…
```

## Proxy Rotation with Crawlee

A proxy acts as a middleman between your computer and the internet, forwarding your web requests while masking your IP address. This helps prevent rate limits and IP bans.

Crawlee simplifies proxy implementation with built-in handling for retries, errors, and rotating proxies.

Next, you'll set up a proxy, obtain a valid proxy address, and verify your requests are routed through it.

Since free proxies are often slow, insecure, and unreliable for sensitive web tasks, consider using a trusted service like Bright Data, which provides secure, stable, and reliable proxies. It also offers free trials, allowing you to test the service before committing. 

To use Bright Data, click the **Start free trial** button on their [home page](https://brightdata.com/) and fill in the required information to create an account.

Once your account is created, log in to the Bright Data dashboard, navigate to **Proxies & Scraping Infrastructure**, and add a new proxy by selecting **[Residential Proxies](/proxy-types/residential-proxies)**:

![Add a residential proxy](https://github.com/luminati-io/crawlee-web-scraping/blob/main/images/Add-a-residential-proxy-1024x574.png)

Retain the default settings and finalize the creation of your residential proxy by clicking **Add**.

If you are asked to install a certificate, you can select **Proceed without certificate**. However, for production and real use cases, you should set up the certificate to prevent misuse if your proxy information is ever exposed.

Once created, take note of the proxy credentials, including the host, port, username, and password. You need these in the next step:

![Bright Data proxy credentials](https://github.com/luminati-io/crawlee-web-scraping/blob/main/images/Bright-Data-proxy-credentials-1024x557.png)

Under the root directory of your project, run the following command to install the [axios](https://www.npmjs.com/package/axios) library:

```bash
npm install axios
```

The `axios` library is used to send a GET request to `http://lumtest.com/myip.json`, which returns details about the proxy in use.

To implement this, create a file named `scrapeWithProxy.js` in your project's root directory and add the following code:

```js
const { CheerioCrawler } = require("crawlee");
const { ProxyConfiguration } = require("crawlee");
const axios = require("axios");

const proxyConfiguration = new ProxyConfiguration({
  proxyUrls: ["http://USERNAME:PASSWORD@HOST:PORT"],
});

const crawler = new CheerioCrawler({
  proxyConfiguration,
  async requestHandler({ request, $, response, proxies }) {
    // Make a GET request to the proxy information URL
    try {
      const proxyInfo = await axios.get("http://lumtest.com/myip.json", {
        proxy: {
          host: "HOST",
          port: PORT,
          auth: {
            username: "USERNAME",
            password: "PASSWORD",
          },
        },
      });
      console.log("Proxy Information:", proxyInfo.data);
    } catch (error) {
      console.error("Error fetching proxy information:", error.message);
    }

    const books = [];
    $("article.product_pod").each((index, element) => {
      const title = $(element).find("h3 a").attr("title");
      const price = $(element).find(".price_color").text();
      books.push({ title, price });
    });
    console.log(books);
  },
});

crawler.run(["https://books.toscrape.com/"]);
```

> **Note:**
> 
> Make sure to replace the `HOST`, `PORT`, `USERNAME`, and `PASSWORD` with your credentials.

This code uses `CheerioCrawler` from `crawlee` to scrape data from `https://books.toscrape.com/` while routing requests through a specified proxy.  

- The proxy is configured using `ProxyConfiguration`.  
- A GET request to `http://lumtest.com/myip.json` fetches and logs proxy details.  
- Book titles and prices are extracted using Cheerio’s jQuery-like syntax and logged to the console.  

Run the code to test the proxy setup and verify its functionality:

```bash
node scrapeWithProxy.js
```

You’ll see similar results to before, but this time, your requests are routed through Bright Data proxies. You should also see the details of the proxy logged in the console:

```js
Proxy Information: {
  country: 'US',
  asn: { asnum: 21928, org_name: 'T-MOBILE-AS21928' },
  geo: {
    city: 'El Paso',
    region: 'TX',
    region_name: 'Texas',
    postal_code: '79925',
    latitude: 31.7899,
    longitude: -106.3658,
    tz: 'America/Denver',
    lum_city: 'elpaso',
    lum_region: 'tx'
  }
}
[
  { title: 'A Light in the Attic', price: '£51.77' },
  { title: 'Tipping the Velvet', price: '£53.74' },
  { title: 'Soumission', price: '£50.10' },
  { title: 'Sharp Objects', price: '£47.82' },
  { title: 'Sapiens: A Brief History of Humankind', price: '£54.23' },
  { title: 'The Requiem Red', price: '£22.65' },
…output omitted..
```

Running the script with `node scrapingWithBrightData.js` should display a different IP address each time, confirming that Bright Data rotates locations and IPs automatically. This rotation helps prevent blockages and IP bans when scraping websites.

> **Note:**
> 
> In the `proxyConfiguration`, you could have passed different proxy IPs, but since Bright Data does that for you, you don’t need to specify the IPs.

## Sessions Management with Crawlee

Sessions help maintain state across multiple requests, especially for sites using cookies or login sessions.  

To implement session management, create a file named `scrapeWithSessions.js` in your project's root directory and add the following code:

```js
const { CheerioCrawler, SessionPool } = require("crawlee");

(async () => {
  // Open a session pool
  const sessionPool = await SessionPool.open();

  // Ensure there is a session in the pool
  let session = await sessionPool.getSession();
  if (!session) {
    session = await sessionPool.createSession();
  }

  const crawler = new CheerioCrawler({
    useSessionPool: true, // Enable session pool
    async requestHandler({ request, $, response, session }) {
      // Log the session information
      console.log(`Using session: ${session.id}`);

      // Extract book data and log it (for demonstration)
      const books = [];
      $("article.product_pod").each((index, element) => {
        const title = $(element).find("h3 a").attr("title");
        const price = $(element).find(".price_color").text();
        books.push({ title, price });
      });
      console.log(books);
    },
  });

  // First run
  await crawler.run(["https://books.toscrape.com/"]);
  console.log("First run completed.");

  // Second run
  await crawler.run(["https://books.toscrape.com/"]);
  console.log("Second run completed.");
})();
```

This code uses `CheerioCrawler` and `SessionPool` from `crawlee` to scrape data from `https://books.toscrape.com/`.  

- A session pool is initialized and assigned to the crawler.  
- The `requestHandler` logs session details and extracts book titles and prices using Cheerio selectors.  
- The script performs two consecutive scraping runs, logging the session ID each time.  

Run the code to verify that different sessions are being used.

```bash
node scrapeWithSessions.js
```

You should see similar results as before, but this time—with the session ID for each run:

```
Using session: session_GmKuZ2TnVX
[
  { title: 'A Light in the Attic', price: '£51.77' },
  { title: 'Tipping the Velvet', price: '£53.74' },
…output omitted…
Using session: session_lNRxE89hXu
[
  { title: 'A Light in the Attic', price: '£51.77' },
  { title: 'Tipping the Velvet', price: '£53.74' },
…output omitted…
```

If you run the code again, you should see that a different session ID is being used.

## Dynamic Content Handling with Crawlee

Scraping **dynamic websites** (those that load content via JavaScript) can be challenging, as data is only available after rendering.  

To handle this, Crawlee integrates with [Puppeteer](https://pptr.dev/), a headless browser that renders JavaScript and interacts with web pages like a human.  

For demonstration, we'll scrape content from [this YouTube page](https://www.youtube.com/watch?v=wZ6cST5pexo). **Before scraping, always review the site's rules and terms of service.**  

After reviewing the terms, create a file named `scrapeDynamicContent.js` in your project's root directory and add the following code:

```js
const { PuppeteerCrawler } = require("crawlee");

async function scrapeYouTube() {
  const crawler = new PuppeteerCrawler({
    async requestHandler({ page, request, enqueueLinks, log }) {
      const { url } = request;
      await page.goto(url, { waitUntil: "networkidle2" });

      // Scraping first 10 comments
      const comments = await page.evaluate(() => {
        return Array.from(document.querySelectorAll("#comments #content-text"))
          .slice(0, 10)
          .map((el) => el.innerText);
      });

      log.info(`Comments: ${comments.join("\n")}`);
    },

    launchContext: {
      launchOptions: {
        headless: true,
      },
    },
  });

  // Add the URL of the YouTube video you want to scrape
  await crawler.run(["https://www.youtube.com/watch?v=wZ6cST5pexo"]);
}

scrapeYouTube();
```

Then, run the code with the following command:

```bash
node scrapeDynamicContent.js
```

This code uses `PuppeteerCrawler` from Crawlee to scrape comments from a YouTube video.  

- The crawler navigates to a specific YouTube video URL and waits for the page to fully load.  
- It selects the first ten comments using the CSS selector `#comments #content-text`.  
- Extracted comments are logged to the console.  

When executed, the script will output the first ten comments from the selected video.

```
INFO  PuppeteerCrawler: Starting the crawler.
INFO  PuppeteerCrawler: Comments: Who are you rooting for?? US Marines or Ex Cons 
Bro Mateo is a beast, no lifting straps, close stance.
ex convict doing the pushups is a monster.
I love how quick this video was, without nonsense talk and long intros or outros
"They Both have combat experience" is wicked 
That military guy doing that deadlift is really no joke.. ...
One lives to fight and the other fights to live.
Finally something that would test the real outcome on which narrative is true on movies
I like the comradery between all of them. Especially on the Bench Press ... Both team members quickly helped out on the spotting to protect from injury. Well done.
I like this style, no youtube funny business. Just straight to the lifts
…output omitted…
```

You can find all the code used in this tutorial on [GitHub](https://github.com/See4Devs/crawlee-web-scraping).

## Conclusion

Crawlee can help improve the efficiency and reliability of your web scraping projects. Ready to elevate your web scraping projects with professional-grade data, tools, and proxies? Explore the comprehensive web scraping platform of Bright Data, offering [ready-to-use datasets](https://brightdata.com/products/datasets) and [advanced proxy services](https://brightdata.com/proxy-types) to streamline your data collection efforts.

Sign up now and start your free trial!
