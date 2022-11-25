# Codelab GDG 2022 - Angular A11y 

These are the steps we follow in the codelab to make our demo Angular app more accessible.

---

## Step 1. Understand what A11y is.

Here we talk about why Accessibility is important.

---
## Step 2. Get the starter code.

You can find the code in under [this repo](https://github.com/IslombekMe/GDG-2022-A11Y).

---
## Step 3. Establish a baseline.

Here we use a series of automatic and manual testing like Lighthouse, axe and VoiceOver to establish what could be improved in our app's a11y.

We also add our custom Lint rules to see warnings in our code before we reach the actual product.

---
## Step 4. Define unique page titles.

Providing unique, concise page titles helps users using a11y services quickly understand a web page's content and purpose. Page titles are critical to users with visual disabilities because they are the first page element announced by screen-readers.

### Identify the issue 
Open three different shop pages and see how they're all titled the same. If you had 20 of such pages, it would be a pain to visit each of them to know what the content is. With Page Titles we can simply give a unique title to a page and already understand what it's about just by looking.

### Solution
Let's add dynamic page titles using Angular 14 Route's new `title` property.

```typescript
const routes: Routes = [
  { path: 'shop', component: ShopComponent, title: 'Our Shop – a11y in Angular' },
  { path: 'about', component: AboutComponent, title: 'Our Story - a11y in Angular' },
  { path: 'locate', component: LocationComponent, title: 'Find Us - a11y in Angular' },
  { path: '',   redirectTo: '/shop', pathMatch: 'full' },
  { path: '**', component: ShopComponent },
];
```

Under the hood it uses [Router's Title Service](https://angular.io/guide/set-document-title#use-the-title-service) to manage changing and for more complex page titles you can implement your own [Title Strategy](https://next.angular.io/api/router/TitleStrategy).

---

## Step 5. Ensure adequate color contrast.

Our designs might seem cool, but it's not if people with visual impairments like color blindness can't read your content. The Web Content Accessibility Guidelines (WCAG 2.0) defines a series of color contrast ratios, that ensure content accessibility.

### Use Chrome Dev Tools to identify low-contrast issues.
1. Using the inspect tool, we can see that the icons' contrast ratio is 1.85 which is below WCAG's >3.0 requirements.
2. Run the Accessibility audit in Lighthouse to see these contrast ratio's.


### Change Material theme color.

**src/styles.scss**
```scss
$light-primary: mat.define-palette(mat.$pink-palette, $default: A100, $lighter: 100, $text: 900);
```

Verifying the changes we see that color has become more crisp and we no longer get warnings from audits.

---

## Step 6. Use Semantic HTML

Running our tests, we can see that there exist a few errors with our HTML. See how headings are not in a sequantial order and . 

**Changing a `<div>` to a `<button>`**

1. **Replace the custom `<div>` with a Material button:**

*src/app/shop/shop.component.html*

```html
<button mat-flat-button 
  color="primary" 
  class="purchase-button"
  (click)="fauxPurchase()">
  Purchase
</button>
```

2. **Reorder the text to use semantic HTML and apply styling using Angular Material typography:**

*src/app/about/about.component.html*
```html
<h2>Who are we?</h2>
<p class="mat-subheading-2">Have you ever thought, "wow, I love dumplings"?</p>
<p class="right mat-subheading-1">Who hasn't.</p>
<p class="center mat-subheading-1">We took it one step further and created Dumpling Dumpling,</p> 
<p class="center mat-subheading-1">double the dumpling, double the fun.</p>
<div class="spacer"></div>
<h2>How are we different?</h2>
<p class="mat-subheading-2">Handmade in San Francisco, California, we craft fully customizable dumplings. Glitter? Rainbows? Vegan? We do it all.</p>
<p class="right mat-subheading-2">This shop is concept only.</p>
```

Having made these changes we already see improvements in our score.

---

## Step 7. Create selectable controls with Angular Material

One complicated interaction pattern for accessibility services is nested controls. Think about menu subitems or nested checkboxes. How do you indicate to a user that you can select a subgroup of options or navigate to a parent menu item?

### Identify the issue
To identify this issue we will turn on our screen reader and attempt to select a nested checkbox.

1. Turn on VoiceOver.
1. Select different filling flavors.
1. Notice that the parent checkboxes don't specify children when read by VoiceOver. How do you know that the Vegan checkbox is unselected now that you unselected the Bok Choy checkbox?

### Solution - A11y in Material

You replace the semantic checkbox with the Angular Material checkbox, which contains built-in knowledge of this interaction pattern. It's important to note that replacing components with Material does not guarantee accessibility. Like any other component, you need to manually test because there are plenty of ways to implement Material inaccessibly.

#### Replace checkboxes with Material checkboxes
1. **First, add your new list of fillings and a variable to store your selected filling flavors:**

*src/app/shop/shop.component.ts*
```typescript
@Component(...)
export class ShopComponent implements OnInit {
  fillings: string[] = ['Bok Choy & Chili Crunch', 'Tofu & Mushroom', 'Chicken & Ginger', 'Impossible Meat & Spinach'];
  selectedFillings: string[] = [];

  fauxPurchase(): void {
    let flavor = '';
    this.selectedFillings.forEach(filling => {
      flavor = flavor + " " + filling
    })
  }
}
```
2. **Add a <mat-selection-list> to replace this messy grouping of HTML checkboxes:**
*src/app/shop/shop.component.html*

```html
<mat-selection-list [(ngModel)]="selectedFillings" 
  aria-label="Dumpling fillings">
  <mat-list-option *ngFor="let flavor of fillings" 
    [value]="flavor" 
    color="primary">
    {{ flavor }}
  </mat-list-option>
</mat-selection-list>
```
Your TODO comments also show where you can remove some unused Sass in *src/app/shop/shop.component.scss* to clean up your styling.

---

## Step 8. Provide control labels with WAI-ARIA

You modified your Angular app's semantic HTML and Material components, but some components require specific attributes to be navigated fully by screen readers.

The [Web Accessibility Initiative's Accessible Rich Internet Applications](https://www.w3.org/TR/wai-aria/) specification (WAI-ARIA or ARIA) helps bridge issues that can't be managed with native HTML. It lets you specify attributes that modify the way an element is translated into the accessibility tree.

### Identify the issue
To identify this issue, turn on your screen reader and move your slider:

1. Turn on VoiceOver.
1. Navigate to the quantity slider and change the value.
1. Notice that the value label is missing.

### Solution - Use ARIA attributes
Label control using aria-label to `<mat-slider>`:

*src/app/shop/shop.component.html*

```html 
<mat-slider
  aria-label="Dumpling order quantity"
  id="quantity"
  name="quantity"
  color="primary"
  class="quantity-slider"
  [max]="13"
  [min]="1"
  [step]="1"
  [tickInterval]="1"
  thumbLabel
  [(ngModel)]="quantity">
</mat-slider>
```

---

## Step 9. Add the power of @angular/cdk/a11y

Add A11yModule in AppModule to introduce all features of Angular CDK's A11y package.

**What does '@angular/cdk/a11y' do?**

The a11y module provides a number of tools to improve accessibility and is specifically useful for component authors.

In the following sections, you add three common services: FocusTrap and LiveAnnouncer.

---

## Step 10. Control focus with FocusTrap

When a dialog or modal is open, a user is interacting only inside it. Allowing the focus to escape outside the dialog mixes contexts and creates a state where the user doesn't know where on the page they are.

In Angular, the cdkTrapFocus directive traps tab-key focus within an element. This is intended to be used to create accessible experience for components like modal dialogs, where focus must be constrained.

(SHOW WHY IT'S NEEDED BY REMOVING FOCUS TRAP ANCHORS IN MAT-DIALOG)

---

## Step 11. Announce changes with LiveAnnouncer

Screen readers need to be notified when something on the page changes. Imagine attempting to submit a form or complete a purchase, and not knowing an error has popped up preventing the form submission. That's frustrating!

LiveAnnouncer is used to announce messages for screen-reader users using an aria-live region to ensure screen readers are notified about notifications and live page changes. For more information on aria-live regions, see the W3C's WAI-ARIA. In Angular, calling LiveAnnouncer as a service is a more testable solution than aria-live attributes.

### Identify the issue

To identify this issue, turn on your screen reader and select Purchase without completing the form fields:

1. Turn on VoiceOver.
1. Use tab navigation to change the color and make a fake purchase.
1. Notice that there's no indication what color was selected when exiting the dialog and the purchase is not read.

### Solution - Add LiveAnnouncer to your code

Add LiveAnnouncer, and announce both color selection and the fake purchase as a string. In a real implementation, this may be read when you navigate to a third-party payment system or for form errors.

1. **Add an announcement when a color is selected:**
*src/app/shop/color-picker/color-picker-dialog/color-picker-dialog.component.ts*

```typescript
import { LiveAnnouncer } from '@angular/cdk/a11y';

@Component(...)
export class ColorPickerDialogComponent implements OnInit {
  constructor(
    public dialogRef: MatDialogRef<ColorPickerDialogComponent>,
    @Inject(MAT_DIALOG_DATA) public data: ColorDialogData,
    private liveAnnouncer: LiveAnnouncer) { }

  public changeColor(color: string): void {
    this.liveAnnouncer.announce(`Select color: ${color}`);
    this.dialogRef.close();
  }
}
```


2. **Add an announcement when a fake purchase is made:**
*src/app/shop/shop.component.ts*

```typescript
import { LiveAnnouncer } from '@angular/cdk/a11y';

@Component(...)
export class ShopComponent implements OnInit {

  constructor(private liveAnnouncer: LiveAnnouncer) { }

  fauxPurchase(): void {
    let flavor = '...';
    const fakePurchase = `Purchase ${this.quantity} ${flavor} dumplings in the color ${this.color}!`;

    this.liveAnnouncer.announce(fakePurchase);
  }
}
```

Turning on screen reader at this point and going thru previously looked at issues now get announced.
