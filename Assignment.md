# Part 1 - receiving data via API

I created following cURL command after refering to the [CA Public JSON APIs](https://customer-alliance.atlassian.net/wiki/spaces/CAA/pages/2620293196/JSON+API+endpoints) documentation. I am passing `start`, `end` and `forced_interval` as query params and passing the API access key in the `X-CA-AUTH` header:

```bash
curl -H 'X-CA-AUTH: 06a7c2a6b2928845f65cee79d8fe01804f032b6dab4dc4b1a648ac7c92651fc3' 'https://api.customer-alliance.com/statistics/v1/reviews-over-time.json?start=2022-01-01&end=2022-03-31&forced_interval=months'
```

Following is the API response:
```bash
{"Urbihop Hotel":{"1 \/ 2022":{"review_count":131,"positive_reviews":105,"neutral_reviews":24,"negative_reviews":2},"2 \/ 2022":{"review_count":93,"positive_reviews":74,"neutral_reviews":19,"negative_reviews":0},"3 \/ 2022":{"review_count":119,"positive_reviews":105,"neutral_reviews":14,"negative_reviews":0}}}
```

Formatted JSON response:
```bash
{
  "Urbihop Hotel": {
    "1 / 2022": {
      "review_count": 131,
      "positive_reviews": 105,
      "neutral_reviews": 24,
      "negative_reviews": 2
    },
    "2 / 2022": {
      "review_count": 93,
      "positive_reviews": 74,
      "neutral_reviews": 19,
      "negative_reviews": 0
    },
    "3 / 2022": {
      "review_count": 119,
      "positive_reviews": 105,
      "neutral_reviews": 14,
      "negative_reviews": 0
    }
  }
}
```

**We can see that reviews are within the provided date range and grouped monthly.**

# Part 2 - checking logs

## Case 1

We can see following message in the log:
`Request contained 0 guests`

Looks like the JSON file uploaded did not contain any guest information. Possible an empty JSON file was uploaded. Hence no guests were uploaded to the system.

## Case 2

From the request payload shown in the logs, we can see that the `gender` attribute of the customers has value `na`. Looks like that the system does not have the gender attribute of the customers (or got deleted due to some reason). This value is most likely refered to form the correct salutation in the emails sent to the customer. Since it is missing, the customers are receiving the emails without the salutation.

## Case 3

From the logs, we can see the information of the 6 guests. Only the guest with last name Frederik has optIns parameter [misc = false]. Thus following guests can be contacted:
Frosh
Johnson
Andersen
Stevens
Brighton

If we further try to interpret from the logs, we can see that some of the guests have canceled their bookings. If this invitation is about reviewing the hotel, we do not want to send the invitation to the guests who have canceled their bookings. Thus we should send the invitation to leave a review only to following guests:
Andersen
Stevens
Brighton
