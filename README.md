# [Crypto Converter Solution](https://frontendeval.com/questions/crypto-converter)

## [Stackblitz playground](https://stackblitz.com/github/willnguyen1312/crypto-converter)

### Constants

- `API_ENDPOINT` - https://api.frontendeval.com/fake/crypto/
- `REFRESH_TIME` - 10000 milliseconds (10 seconds)

### State

- amount: `number` - The amount of the currency to convert.
- currency: `string` - The currency to convert from.
- lastValue: `number` - The last computed value of the conversion.
- currentValue: `number` - The current computed value of the conversion.

### Rendering

Simple form with two inputs of amount and currency. The amount input is a number input and the currency input is a select input with the options of the currencies

The result is displayed when there are valid values in both inputs

The changed value is displayed when there are enough information about last and current computed values to calculate the difference

Everything is rendered in the center horizontally by using flexbox

### Crypto conversion logic

- `fetchValue`: The conversion is done by using the `fetch` API to get the conversion rate from the API endpoint. Then update the last value and current value in the state.

- `intervalId`: The interval is set up by using `setInterval` to call `fetchValue` every `REFRESH_TIME` milliseconds. This will be cleaned on up unmount lifecycle to avoid memory leak.

- There is a single watcher to watch the changes of `amount` and `currency` in the state. When there are changes in either of them, lastValue and currentValue will be reset and the `fetchValue` will be called to fetch the new conversion rate. The current interval will be cleared if available and a new interval will be set up.

- There are two computed values `changedValue` and `displayChangedValue` to avoid clustering the template with logic. `changedValue` is the difference between lastValue and currentValue. `displayChangedValue` is the formatted value of `changedValue` with the up / down symbols. The `changedValue` is also used to determine the color of the result.

### Nice to have

- [ ] Add a loading indicator
- [ ] Add a refresh button
- [ ] Add a error message when the API call fails
