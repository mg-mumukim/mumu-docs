| Key           | Value                                                                                    |
| ------------- | ---------------------------------------------------------------------------------------- |
| Source        | https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U/edit?tab=t.qtynrelslsnw |
| Downloaded at | 2026-04-22 10:20:48 |

---

# Backend Contract

Maintainer [Brandon Choi](mailto:brandon.choi@match.com)  
Reviewer [Mumu Kim](mailto:mumu.kim@match.com) [Trideep Rath](mailto:trideep.rath@gotinder.com)  
Last updated 2026-03-24  
---

[POST /v1/photo-review](#post-/v1/photo-review)

[Field Priority](#field-priority)

[Request Body](#request-body)

[Tie-Breaker Metric for Repetitive Photos](#tie-breaker-metric-for-repetitive-photos)

[Request Body](#request-body-1)

[Response Body (200)](#response-body-\(200\))

[Option 1: Return all variants at once and let the client or backend choose which variant to use](#option-1:-return-all-variants-at-once-and-let-the-client-or-backend-choose-which-variant-to-use)

[Option 2: Reduce duplicated fields from option 1](#option-2:-reduce-duplicated-fields-from-option-1)

[Option 3: Add photo insight lever argument to the request and returns only relevant cards](#option-3:-add-photo-insight-lever-argument-to-the-request-and-returns-only-relevant-cards)

[Response Body (4XX/5XX)](#response-body-\(4xx/5xx\))

---

# POST /v1/photo-review {#post-/v1/photo-review}

## Field Priority {#field-priority}

**Note:** ML team is exploring causal analysis by 4/10. Some P1 fields may be promoted to P0 as ML requirements solidify.

### Request Body {#request-body}

| Field | Priority | Required | Reasoning |
| :---- | :---- | :---- | :---- |
| `request_id` | P0 | Yes | Request tracing and idempotency |
| `age` | P0 | Yes | KB atom segment matching (age group bucketing) \+ prompt context |
| `gender` | P0 | Yes | KB atom segment matching \+ prompt context |
| `target_gender` | P1 | No | Not used in current coaching pipeline. May be needed for future atom matching. **Should be an array** (users can be interested in multiple genders) |
| `country_code` | P0 | Yes | KB atom segment matching (ISO 3166-1 alpha-2) |
| `city` | P1 | No | Included in prompt context but does not materially affect coaching quality |
| `job_titles` | P1 | No | Personalization fuel — LLM references for tailored suggestions. Coaching works without it |
| `employers` | P1 | No | Same as above |
| `schools` | P1 | No | Same as above |
| `languages` | P1 | No | Not used currently. Needed for future multi-language coaching |
| `anthem` | P1 | No | Personalization fuel. Most users don't set this |
| `interests` | P0 | Yes | Core to photo suggestion personalization (e.g. "Add a photo of you hiking") |
| `lifestyles` | P1 | No | Personalization fuel. Helpful but not blocking |
| `basics` | P1 | No | Same as above |
| `relationship_intent` | P1 | No | Not used in current pipeline. May influence tone in future |
| `bio` | P0 | Yes | Coaching prompt behavior differs based on bio presence (empty bio → photo-only focus) |
| `photos` | P0 | Yes | Primary input. Minimum 1 photo required |
| `photos[].id` | P0 | Yes | Used in response to identify repetitive photos |
| `photos[].bucket` | P0 | Yes | S3 image loading |
| `photos[].s3_key` | P0 | Yes | S3 image loading |
| `photos[].smartphoto_score` | **TBD** | **TBD** | See "Tie-Breaker Metric" below |

### Tie-Breaker Metric for Repetitive Photos {#tie-breaker-metric-for-repetitive-photos}

When repetitive/duplicate photos are detected, the system must select a **representative** (the photo to keep) from each cluster. This requires a ranking metric.

Currently available options (`smartphoto_score`, `top_impressions/top_likes`) have limited coverage across the user population, making them unreliable as a universal tie-breaker.

**Request to Tinder team:** Please suggest an alternative photo-level quality metric that is available for **all users** (e.g. photo quality score, face detection confidence, recency/upload timestamp, or any internal ranking signal). The metric does not need to be highly precise — it only needs to be directionally correct for choosing the stronger photo within a visually similar cluster.

If no universal metric exists, the fallback is LLM-based visual quality judgment, but this is harder to validate at scale.

## Request Body {#request-body-1}

```json
{
    "request_id": "71639584-1234-5678-9012-345678901234",

    // --- P0: cohort for branching logic ---
    "age": 21,
    "gender": 0,
    "country_code": "US",
    // FIXME: We need smartphoto on/off info and smartphoto version info

    // --- P1: cohort (optional) ---
    "target_gender": [0, 1],
    "city": "New York",

    // --- P1: personal information (optional, personalization fuel) ---
    "job_titles": ["Software Engineer", "Backend Software Engineer"],
    "employers": ["Tinder"],
    "schools": ["MIT", "Stanford", "Harvard", "Georgia Tech"],
    "languages": ["English", "Spanish"],
    "anthem": { "name": "Faded", "artists": "Alan Walker" },
    "lifestyles": [
        { "name": "Pets", "value": "Dog" },
        { "name": "Drinking", "value": "Most Nights" },
        { "name": "Smoking", "value": "Non-Smoker" },
        { "name": "Workout", "value": "Never" },
        { "name": "Dietary Preference", "value": "Vegetarian" },
        { "name": "Sleeping Habits", "value": "Night Owl" }
    ],
    "basics": [
        { "name": "Zodiac", "value": "Virgo" },
        { "name": "Education", "value": "Bachelor's Degree" },
        { "name": "COVID vaccine", "value": "Vaccinated" },
        { "name": "MBTI", "value": "INTP" },
        { "name": "Family Plans", "value": "Want kids" },
        { "name": "Communication Style", "value": "Big time texter" }
    ],
    "relationship_intent": "Long-term, open to short",
    "interests": ["Travel", "Hiking", "Coffee"],

    // --- P0: unstructured contents ---
    "bio": "I'm a software engineer at Tinder. You LGTM.",
    "photos": [ // This should be in original photo position order
        {
            "id": "528ab123-4567-8901-2345-678901234567",
            "bucket": "tinder-profile-photos-prod",
            "s3_key": "images/dt=2026-03-23/528ab123-4567-8901-2345-678901234567.jpg",
            // TBD: tie-breaker metric — pending Tinder team input
            "smartphoto_score": 0.85
        },
        {
            "id": "4c3f8123-4567-8901-2345-678901234567",
            "bucket": "tinder-profile-photos-prod",
            "s3_key": "images/dt=2026-03-23/4c3f8123-4567-8901-2345-678901234567.jpg",
            "smartphoto_score": 0.5
        }
    ]
}

```

## Response Body (200) {#response-body-(200)}

### Option 1: Return all variants at once and let the client or backend choose which variant to use {#option-1:-return-all-variants-at-once-and-let-the-client-or-backend-choose-which-variant-to-use}

```json
{
    "request_id": "71639584-1234-5678-9012-345678901234",
    "status": "success",
    "model_name": "gemini/gemini-3.1-pro-preview",
    "prompt_name": "photo-coaching-2026-03-17",

    "photo_insight_card": [
        {
            "variant_name": "connect",
            "title": "Mix up your photo angles",
            "body": "Adding variety beyond selfies helps people get a better read on you and feel comfortable connecting."
        },
        {
            "variant_name": "conversation",
            "title": "Mix up your photo angles",
            "body": "Adding a hobby or social shot gives people an easy way to start a conversation."
        },
        {
            "variant_name": "compatibility",
            "title": "Mix up your photo angles",
            "body": "Showing your interests instead of just selfies makes shared interests easy to spot."
        },
        {
            "variant_name": "personality",
            "title": "Mix up your photo angles",
            "body": "Swapping a selfie for a full-body or hobby shot shows more of who you are."
        },
        {
            "variant_name": "stand_out",
            "title": "Mix up your photo angles",
            "body": "A diverse mix of photos helps you stand out naturally."
        }
    ],
    "action_card": {
        "title": "Mix up your photo angles",
        "body": "Swap the filtered selfie for a full-body shot. Try adding a photo showing you baking or enjoying coffee."
    },
    "repetitive_photos": [ "4c3f8123-4567-8901-2345-678901234567" ]
}
```

### Option 2: Reduce duplicated fields from option 1 {#option-2:-reduce-duplicated-fields-from-option-1}

```json
{
    "request_id": "71639584-1234-5678-9012-345678901234",
    "status": "success",
    "model_name": "gemini/gemini-3.1-pro-preview",
    "prompt_name": "photo-coaching-2026-03-17",

    "card_title": "Mix up your photo angles",
    "card_bodies": [
        {
            "name": "photo_insight_card/connect",
            "body": "Adding variety beyond selfies helps people get a better read on you and feel comfortable connecting."
        },
        {
            "name": "photo_insight_card/conversation",
            "body": "Adding a hobby or social shot gives people an easy way to start a conversation."
        },
        {
            "name": "photo_insight_card/compatibility",
            "body": "Showing your interests instead of just selfies makes shared interests easy to spot."
        },
        {
            "name": "photo_insight_card/personality",
            "body": "Swapping a selfie for a full-body or hobby shot shows more of who you are."
        },
        {
            "name": "photo_insight_card/stand_out",
            "body": "A diverse mix of photos helps you stand out naturally."
        },
        {
            "name": "action_card",
            "body": "Swap the filtered selfie for a full-body shot. Try adding a photo showing you baking or enjoying coffee."
        }
    ],
    "repetitive_photos": [ "4c3f8123-4567-8901-2345-678901234567" ]
}

```

### Option 3: Add photo insight lever argument to the request and returns only relevant cards {#option-3:-add-photo-insight-lever-argument-to-the-request-and-returns-only-relevant-cards}

```json
{
    "request_id": "71639584-1234-5678-9012-345678901234",
    "status": "success",
    "model_name": "gemini/gemini-3.1-pro-preview",
    "prompt_name": "photo-coaching-2026-03-17",

    // assume the request contains `"photo_insight_variant": "connect"`
    "photo_insight_card": {
        "title": "Mix up your photo angles",
        "body": "Adding variety beyond selfies helps people get a better read on you and feel comfortable connecting."
    },
    "action_card": {
        "title": "Mix up your photo angles",
        "body": "Swap the filtered selfie for a full-body shot. Try adding a photo showing you baking or enjoying coffee."
    },
    "repetitive_photos": [ "4c3f8123-4567-8901-2345-678901234567" ]
}
```

## Response Body (4XX/5XX) {#response-body-(4xx/5xx)}

```json
{
    "request_id": "71639584-1234-5678-9012-345678901234",
    "status": "failure",
    "model_name": "gemini/gemini-3.1-pro-preview",
    "prompt_name": "2026-03-17/v1",

    // I did not think about this much
    "error": {
        "code": "<error code for future automation>",
        "location": "<optional `jsonpath` for the request body that caused the error>",
        "description": "<human-readable description of the error>"
    }
}
```

