# Data Dictionary

Field reference for the three GA4-derived datasets used across the notebooks in this
repository. Raw data is not included; this file documents the fields so the notebooks are readable without the
original CSVs.

## 1. Time to Purchase dataset (`parta_01_time_to_purchase.ipynb`)
Event-level log covering ~2 months of traffic, restricted to users who completed at least
one purchase (298,407 rows · 911 users · 5,230 sessions).

**Identifiers & user context**
| Field | Description |
|---|---|
| `user_pseudo_id` | Anonymous user identifier (cookie ID, device/browser level) |
| `user_id` | Logged-in user identifier (if available) |
| `session_id` | Session identifier: unique for the individual, not globally unique |
| `ga_session_number` | Session count for the user (journey depth) |
| `login_status` | Login state (logged-in or guest) |
| `country_long` | User's country |
| `device_category` / `device_model` | Device type (mobile/desktop) and model |

**Event context**
| Field | Description |
|---|---|
| `event_date` / `event_timestamp` / `event_timestamp_hms` | Date and raw/human-readable timestamp of the event |
| `event_name` | Type of event (e.g. `view_item`, `add_to_cart`, `purchase`) |
| `page_location` | Page URL visited |
| `search_term` | On-site search query |
| `engagement_time_msec` | User engagement time (ms) |

**Product & transaction**
| Field | Description |
|---|---|
| `item_id` / `item_category` | Product identifier and category |
| `item_price` / `item_quantity` / `item_revenue` | Unit price, quantity, and revenue for the item |
| `cart_value_estimate` | Estimated cart value - identical to `item_price` on `view_item` rows, so not used as a standalone feature |
| `transaction_id` | Transaction identifier (order ID) |

## 2. Cart Abandonment dataset (`parta_02_cart_abandonment.ipynb`)
Session-level table (one row per user × session), aggregated from the raw event log
(212,656 sessions).

**Identifiers & user context**
| Field | Description |
|---|---|
| `user_pseudo_id` / `session_id` / `user_id` | User and session identifiers |
| `login_status` | Login state (logged-in or guest) |
| `ga_session_number` | Session sequence number (journey depth) |
| `event_date` | Date of the session |
| `time_of_day_segment` | Time-of-day bucket (morning / afternoon / evening / night) |

**Funnel flags**
| Field | Description |
|---|---|
| `viewed_product` / `added_to_cart` / `started_checkout` / `purchased` | Binary indicator (0/1) that the session reached each funnel stage |
| `drop_off_stage` | Furthest stage reached before drop-off: one of `no_product_interaction`, `product_abandonment`, `cart_abandonment`, `checkout_abandonment`, `converted` |

**Item & cart counts**
| Field | Description |
|---|---|
| `items_viewed` / `items_added_to_cart` | Total item view / add-to-cart events in the session |
| `unique_items_viewed` /<br>`unique_items_added_to_cart` /<br>`unique_items_purchased` | Distinct item counts per stage |
| `unique_categories_viewed` | Distinct product categories viewed |
| `checkout_events` | Count of checkout-related events |
| `cart_value` / `avg_viewed_price` | Total cart value / average price of viewed items |

**Session engagement**
| Field | Description |
|---|---|
| `total_engagement_time` | Summed on-page engagement time for the session (ms) |
| `session_duration_sec` | Total session length in seconds |

**Funnel timestamps**: each funnel stage has the same five time representations
| Metric | First view | First cart add | First checkout | Purchase |
|---|---|---|---|---|
| Raw timestamp | `first_view_ts` | `first_cart_ts` | `first_checkout_ts` | `purchase_ts` |
| Datetime | `first_view_datetime` | `first_cart_datetime` | `first_checkout_datetime` | `purchase_datetime` |
| Date | `first_view_date` | `first_cart_date` | `first_checkout_date` | `purchase_date` |
| Time | `first_view_time` | `first_cart_time` | `first_checkout_time` | `purchase_time` |
| Hour (0–23) | `first_view_hour` | `first_cart_hour` | `first_checkout_hour` | `purchase_hour` |

**Funnel timing** (elapsed time between stages)
| Field | Description |
|---|---|
| `time_view_to_cart` | First view → first cart add |
| `time_cart_to_checkout` | First cart add → checkout start — the strongest predictor of purchase in the Stage 2 model |
| `time_checkout_to_purchase` | Checkout start → purchase |

## 3. Market Basket Analysis dataset (`partb_market_basket_analysis.ipynb`)
Transaction-level (purchase) data (44,376 rows · 35,109 transactions · 32,119 users ·
3,466 unique items · 36 commercial categories).

**Identifiers**
| Field | Description |
|---|---|
| `event_date` | Date of the purchase |
| `user_pseudo_id` / `session_id` | User and session identifiers |
| `country` | User's country |
| `transaction_id` | Transaction identifier (order ID) |

**Product attributes**
| Field | Description |
|---|---|
| `item_id` / `item_category` | Product identifier and category |
| `style_fabric_color` | Product attributes combining style, fabric, and color |
| `operative_brand` | Brand associated with the product (brand grouping) |
| `commercial_category_group` /<br>`commercial_category` /<br>`commercial_sub_category` | High → mid → detailed product classification |

## Key Engineered Features
Derived variables computed in the notebooks, not present in the raw source data.

**Time to Purchase**
| Field | Description |
|---|---|
| `session_item_sim` | Average pairwise cosine similarity of OpenAI `text-embedding-3-small` embeddings for items viewed in a session - proxy for focused vs. scattered browsing |
| `channel` | Derived UTM-based traffic channel (organic / email / paid_search / shopping / social / …) |
| `days_to_purchase` | Days between a user's first item view and eventual purchase (survival analysis target) |
| `purchase_in_session` | Binary target: did this session end in a purchase |
| `relationship` | LLM-classified relationship between a viewed item pair (complementary / substitute / style_match / unrelated) |
| `later_copurchased` | Binary target: were the two viewed items purchased together in a later session |

**Cart Abandonment**
| Field | Description |
|---|---|
| `target_purchase` | Binary target for the Stage 2 model: did this cart session end in a purchase |
| `stage1_prob` | Predicted probability of add-to-cart (Stage 1 output) |
| `stage2_prob` | Predicted probability of purchase given cart (Stage 2 output) |
| `combined_prob` | Full-funnel purchase score, `stage1_prob × stage2_prob` |

**Market Basket Analysis**
| Field | Description |
|---|---|
| `product_label` | Human-readable product label built from `item_id`, `item_category`, `style_fabric_color`, `operative_brand`, and category fields, used as the item identifier in association-rule mining |
