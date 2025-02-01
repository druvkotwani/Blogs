## What is Cookie Consent a Test ?

A cookie consent banner informs users about the use of cookies on your website and requests their consent to collect and process their data.

## Why it is needed?

- **Legal Compliance**: Required by regulations like GDPR (General Data Protection Regulation) and CCPA (California Consumer Privacy Act). EU, America
- **Transparency**: Increases trust by informing users about data collection.

## **Why are there multiple options in the banner?**

- To allow users to **choose their preferences**, such as:
    - **Analytics Cookies**: For tracking website performance.
    - **Essential Cookies**: Required for the website to function.
    - **Marketing Cookies**: For personalized ads.

## How Does a Cookie Consent Banner Work?

1. **Initial Display**: Shows the banner when a user visits the website for the first time.
2. **User Interaction**: User selects options (Accept All, Reject All, or Customize).
3. **Save Preferences**: Save the user's preferences in local storage or a cookie.
4. **Conditional Logic**: Load cookies based on the user's consent.

## **Understanding the Process**

1. **Display the Banner**:
    - Show a banner when a user visits the website for the first time.
    - The banner will have a message about cookie usage and an option to accept cookies.
2. **Store Consent**:
    - Once the user clicks "Accept," store the consent information in a **cookie** for 30 days using a library like `js-cookie`.
3. **Avoid Preferences**:
    - In this simplified version, no need for multiple cookie preferences (essential, analytics, etc.).
    - We’ll store a single flag (`cookieConsent`) with a value of `true`.
4. **Storage**:
    - Use **browser cookies** instead of `localStorage`:
        - **Why cookies?** Cookies can be accessed by both the client (browser) and server (during SSR/ISR).
        - **TTL (Time to Live):** Cookies can be easily set to expire after 30 days.
        - **Why not `localStorage`?** It’s accessible only on the client-side, making it less versatile for SSR.
5. **TTL for Cookies**:
    - Cookies can have a specific expiration date (e.g., 30 days). This is straightforward to implement with libraries like `js-cookie`.

### **What Are Cookies?**

Cookies are like **sticky notes** that both **you (the browser)** and **the website's server** can read. They are small pieces of information sent with every request to the server.

- Cookies can be set to **expire** (like in 30 days).
- They are **accessible to both the browser and the server**.

### **Why Use Cookies for SSR in Next.js?**

In Next.js apps:

- If you store user preferences in **local storage**, the server can’t read them.
- If you store them in **cookies**, the server can read the preferences and **customize the page** accordingly.

For example:

1. In a cookie consent banner:
    - If you store user consent in **local storage**, the server will still show the banner on the next visit (because it doesn’t know about the consent).
    - If you store consent in a **cookie**, the server will know the user has already accepted and **hide the banner**.

### **Comparison Table: Local Storage vs Cookies**

| Feature | **Local Storage** | **Cookies** |
| --- | --- | --- |
| **Where it's stored** | Only in your browser | In both browser and server |
| **Accessible by Server** | No | Yes |
| **Expiration** | No expiration unless cleared | Can set expiration (e.g., 30 days) |
| **Size Limit** | Up to ~5MB | Smaller (~4KB per cookie) |
| **Best for** | Storing app settings locally | Handling user sessions, preferences, SSR |

### **Steps to Implement Cookie Consent in Next.js 13**

### 1. **Install `js-cookie`**

Install a library to manage cookies easily:

```bash
npm install js-cookie
```

---

### 2. **Create the Cookie Consent Banner Component**

Here’s the code for a minimal cookie consent banner:

```jsx
'use client';

import { useState, useEffect } from 'react';
import Cookies from 'js-cookie';

const CookieConsentBanner = () => {
  const [showBanner, setShowBanner] = useState(false);

  useEffect(() => {
    const consent = Cookies.get('cookieConsent');
    if (!consent) {
      setShowBanner(true);
    }
  }, []);

  const handleAcceptCookies = () => {
    Cookies.set('cookieConsent', 'true', { expires: 30 }); // Set cookie for 30 days
    setShowBanner(false);
  };

  if (!showBanner) return null;

  return (
    <div style={bannerStyles}>
      <p>We use cookies to improve your experience. By using this site, you agree to our use of cookies.</p>
      <button onClick={handleAcceptCookies}>Accept</button>
    </div>
  );
};

const bannerStyles = {
  position: 'fixed',
  bottom: 0,
  left: 0,
  right: 0,
  background: '#f8f8f8',
  padding: '15px',
  borderTop: '1px solid #ccc',
  display: 'flex',
  justifyContent: 'space-between',
  alignItems: 'center',
  zIndex: 1000,
};

export default CookieConsentBanner
```

---

### 3. **Integrate the Component into the Layout**

Include the `CookieConsentBanner` in your global layout so it appears across the website:

```jsx
import CookieConsentBanner from './components/CookieConsentBanner';

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        {children}
        <CookieConsentBanner />
      </body>
    </html>
  );
}
```

---

### 4. **Validate Cookie Storage**

After the user accepts cookies:

- The `cookieConsent` cookie will be created.
- The cookie will expire after 30 days.
    
    You can validate this by:
    
    1. Opening the browser’s developer tools.
    2. Navigating to the **Application tab**.
    3. Checking the **Cookies section** under your domain.

## **Storing Cookies vs. Local Storage**

### **Why Use Cookies?**

1. **Server-Side Access**:
    - Cookies can be accessed during server-side rendering (SSR) in Next.js.
    - This is useful for deciding server-rendered content based on consent.
2. **Expiration**:
    - Cookies can automatically expire after a specified period (e.g., 30 days).
3. **HTTP Support**:
    - Cookies are sent with HTTP requests, allowing the server to use them for session tracking.

### **Why Not Use Local Storage?**

1. **Client-Side Only**:
    - Local storage is accessible only on the client side.
    - SSR and API requests cannot use local storage data.
2. **No Expiration**:
    - Local storage data persists indefinitely unless explicitly cleared by JavaScript.

### **How This Works in Practice**

1. **First Visit**:
    - The banner checks if the `cookieConsent` cookie exists.
    - If not, the banner is displayed.
2. **User Accepts**:
    - When the user clicks "Accept," the `cookieConsent` cookie is created with a 30-day TTL.
    - The banner disappears.
3. **Subsequent Visits**:
    - If the `cookieConsent` cookie is present, the banner does not show.

---

### **What is Local Storage?**

Think of **local storage** like a **notepad on your computer** that only **you** can see and access. It’s used by your browser (like Chrome) to store data **locally** on your device.

- It **doesn’t expire** unless you manually clear it.
- It’s **only available in the browser**, not accessible to the server.

---

### **Local Storage Example**

Here’s an example of using local storage in JavaScript:

```jsx
javascript
Copy code
// Save something in local storage
localStorage.setItem('userConsent', 'true');

// Retrieve it later
const userConsent = localStorage.getItem('userConsent');
console.log(userConsent); // Output: 'true'

// Remove it
localStorage.removeItem('userConsent');

```

---

### **Problem with Local Storage in SSR**

In **Next.js**, **SSR (server-side rendering)** means some of the code runs on a server before it sends the page to your browser.

Since **local storage is only in the browser**, when Next.js runs on the server, it can’t access local storage.

Imagine this:

1. The server renders a page, but it doesn’t know what’s in your local storage.
2. So, if you saved some settings in local storage (like cookie preferences), the server won’t know and can’t adjust the page accordingly.

---

### **What Are Cookies?**

Cookies are like **sticky notes** that both **you (the browser)** and **the website's server** can read. They are small pieces of information sent with every request to the server.

- Cookies can be set to **expire** (like in 30 days).
- They are **accessible to both the browser and the server**.

---

### **Cookie Example**

Here’s a simple example of setting, getting, and deleting cookies using **JavaScript**:

```jsx
javascript
Copy code
// Set a cookie that expires in 30 days
document.cookie = "userConsent=true; max-age=2592000"; // 30 days in seconds

// Get all cookies
console.log(document.cookie); // Output: 'userConsent=true'

// Delete a cookie (set max-age to 0)
document.cookie = "userConsent=; max-age=0";

```

---

### **How Cookies Work with SSR**

Because cookies are sent with every request to the server:

- **When you visit a website**, the server can read your cookies and adjust the page based on your preferences.

Example:

1. You visit a site and accept the cookie policy.
2. The server stores your preference in a cookie.
3. Next time you visit, the server reads the cookie and decides **not to show** the cookie banner.
