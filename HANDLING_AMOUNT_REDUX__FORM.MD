## Handling amount input with redux-form

To ensure that the input field has a valid amount on each input change, we should use the `formatMoney` method which is actually a mask that restricts the entered values.

```
  formatMoney('12d') -> '12'
  formatMoney('12.') -> '12.'
  formatMoney('$123.4') -> '123.4'
  formatMoney('123.456') -> '123.45'
  formatMoney('1,223.45') -> '1223.45'
```

We also want the currency symbol next to the amount, to do that we use the `addCurrencyMethod`, which is (curried).
  ```
  addCurrency('USA')('123') -> '$ 123'
 ```

We can compose this functions together using the `compose` method provided by the redux library

```
const formatAmountInput = compose(addCurrency(providerCurrency), formatMoney)
```

which is exactly the same thing as doing:

```
const formatAmountInput = (amount) => {
  return addCurrency(providerCurrency, formatMoney(amount))
}
```

We can use this composition and get the following results:
```
// Assuming providerCurrency = 'USA'
formatAmountInput(12d) -> $ 12
formatAmountInput(123.456) -> $ 123.45
```

#### Where do we use this function?
On the `normalize` prop of the `Field` you want to mask:
```
<Field normalize={(value) => formatAmountInput(value)} name="amount" component="input" />
```

The function passed to the normalize prop is called each time the input is updated with a new value, the returned value will be the one stored in the store. Then, the `Field` will use that value to re-render the input.

We can also want to have the `priceInCents` value as a form field too. To do that, we use the `change` function provided as a prop by the redux-form HOC.
This function let us change the current field values in the store. So whenever the `amount` field changes we can call: 
```
change('priceInCents', transformToCents(value))
```
Remember that `value` is the current value of the `amount` field. So this `transformToCents` should actually:
 * Remove the currency: ```$ 123.2 -> 123.2```
 * Multiply by 100 to get the cents: ```123.2 -> 12320```

We have functions to do that in our formatters.js: 
`removeCurrency`, and `formatToCents`.

we can compose them too:
```
const transformToCents = compose(formatToCents, removeCurrency)
```
Then we put everything together in a normalizer function:
```
const normalizeHandler =  = (value) => {
  change("priceInCents", transformToCents(value))
  return formatAmountInput(value)
}
```
and add it as the normalize prop:
```
<Field normalize={normalizeHandler} name="amount" component="input" />
```
