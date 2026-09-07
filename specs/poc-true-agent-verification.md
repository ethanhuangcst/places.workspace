# POC true-agent verification — Lisbon 4-day

**Date:** 2026-09-07T02:30:39.091Z  
**Verdict:** PASS — ready for human review  
**Elapsed:** 43 ms  
**Scenario:** Lisbon 4-day trip from Hills Hotel Lisbon (2026-09-12 → 2026-09-15)

## Constraints (scenario)

| Key | Value |
| --- | --- |
| Destination | Lisbon |
| Dates | 2026-09-12 → 2026-09-15 |
| Days | 4 |
| Party size | 2 (report-only; not on PlanTripInput) |
| Origin | Hills Hotel Lisbon |
| Budget | premium |
| Trip type | couple |
| Transit | public_transit |
| Daily start | 07:00 (report-only; fill mock slot) |
| Must-see candidates (5) | Torre de Belém, Mosteiro dos Jerónimos, Castelo de São Jorge, LX Factory, Miradouro da Senhora do Monte |
| Must include (3) | Torre de Belém, Mosteiro dos Jerónimos, Castelo de São Jorge |

## Environment

| Key | Value |
| --- | --- |
| `PLACES_VENDOR_MODE` | `fixture` |
| `PLAN_TRIP_LEGACY_FULL_LOOP` | `(unset)` |
| Google / live adapters | not invoked (`_testGeocode` / `_testSearchPlaces` injectors + fixture mode) |
| LLM keys | cleared in-process |
| Stops pool store | Prisma `AttractionPoi` (Lisbon seed from prior live run; this script did not call Google) |
| Days / origin | 4 / Hills Hotel Lisbon |

## Checks

| ID | Result | Detail |
| --- | --- | --- |
| vendor_fixture | PASS | PLACES_VENDOR_MODE=fixture |
| no_legacy_pipeline | PASS | PLAN_TRIP_LEGACY_FULL_LOOP unset (agent loop default) |
| status_ready | PASS | status=ready |
| skeleton_days | PASS | days=4 (expect 4) |
| filled_stops | PASS | filledStops=24 (expect ≥4) |
| must_include | PASS | must_include=Torre de Belém, Mosteiro dos Jerónimos, Castelo de São Jorge |
| tool_order | PASS | resolve_origin_stay → search_places → make_itinerary → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → commit_artifacts (plan_next_stop×24) |
| registry_chips | PASS | poolAfter=211 poolBefore=211 |
| no_live_keys | PASS | QWEN/OPENAI keys cleared for this process |
| no_time_overlap | PASS | no overlap |
| has_afternoon | PASS | all 4 days have ≥1 stop starting ≥13:00 |
| meal_has_card | PASS | all meal stops have restaurant card |

## `tool_calls`

Intake + full-loop sequence:

```
geocode → search_places → commit_trip → resolve_origin_stay → search_places → make_itinerary → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → commit_artifacts
```

Model-chosen full-loop tools (must match scripted act-or-stop, not a hidden fixed pipeline name):

```
resolve_origin_stay → search_places → make_itinerary → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → plan_next_stop → commit_artifacts
```

## Skeleton

```json
[
  {
    "day_index": 1,
    "day_theme": "Belém waterfront",
    "stops": [
      {
        "name": "Hills Hotel Lisbon",
        "kind": "stay"
      },
      {
        "name": "Torre de Belém",
        "kind": "attraction"
      },
      {
        "name": "Mosteiro dos Jerónimos",
        "kind": "attraction"
      },
      {
        "name": "lunch",
        "kind": "meal"
      },
      {
        "name": "Miradouro da Senhora do Monte",
        "kind": "attraction"
      },
      {
        "name": "dinner",
        "kind": "meal"
      }
    ]
  },
  {
    "day_index": 2,
    "day_theme": "Alfama & castle",
    "stops": [
      {
        "name": "Hills Hotel Lisbon",
        "kind": "stay"
      },
      {
        "name": "Castelo de São Jorge",
        "kind": "attraction"
      },
      {
        "name": "Miradouro da Senhora do Monte",
        "kind": "attraction"
      },
      {
        "name": "lunch",
        "kind": "meal"
      },
      {
        "name": "LX Factory",
        "kind": "attraction"
      },
      {
        "name": "dinner",
        "kind": "meal"
      }
    ]
  },
  {
    "day_index": 3,
    "day_theme": "Alcântara creative",
    "stops": [
      {
        "name": "Hills Hotel Lisbon",
        "kind": "stay"
      },
      {
        "name": "LX Factory",
        "kind": "attraction"
      },
      {
        "name": "Torre de Belém",
        "kind": "attraction"
      },
      {
        "name": "lunch",
        "kind": "meal"
      },
      {
        "name": "Mosteiro dos Jerónimos",
        "kind": "attraction"
      },
      {
        "name": "dinner",
        "kind": "meal"
      }
    ]
  },
  {
    "day_index": 4,
    "day_theme": "Return to Belém",
    "stops": [
      {
        "name": "Hills Hotel Lisbon",
        "kind": "stay"
      },
      {
        "name": "Torre de Belém",
        "kind": "attraction"
      },
      {
        "name": "Mosteiro dos Jerónimos",
        "kind": "attraction"
      },
      {
        "name": "lunch",
        "kind": "meal"
      },
      {
        "name": "Castelo de São Jorge",
        "kind": "attraction"
      },
      {
        "name": "dinner",
        "kind": "meal"
      }
    ]
  }
]
```

## filledStops

```json
[
  {
    "day_index": 1,
    "stop_index": 0,
    "stop": {
      "name": "Hills Hotel Lisbon",
      "kind": "stay",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "07:00",
      "end": "07:30"
    }
  },
  {
    "day_index": 1,
    "stop_index": 1,
    "stop": {
      "name": "Torre de Belém",
      "kind": "attraction",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "08:00",
      "end": "09:30"
    }
  },
  {
    "day_index": 1,
    "stop_index": 2,
    "stop": {
      "name": "Mosteiro dos Jerónimos",
      "kind": "attraction",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "10:00",
      "end": "11:30"
    }
  },
  {
    "day_index": 1,
    "stop_index": 3,
    "stop": {
      "name": "Time Out Market Lisboa",
      "kind": "meal",
      "card": {
        "provider": "GOOGLE_MAPS",
        "name": "Time Out Market Lisboa",
        "location": {
          "lat": 38.71,
          "lng": -9.14,
          "crs": "WGS84"
        },
        "category": "restaurant",
        "photos": [
          "https://cdn.example.com/verify_meal_lunch_0.jpg"
        ],
        "sources": [
          {
            "provider": "GOOGLE_MAPS",
            "native_id": "verify_meal_lunch_0",
            "deeplinks": {}
          }
        ]
      },
      "deeplinks": {}
    },
    "slot": {
      "start": "12:00",
      "end": "13:00"
    }
  },
  {
    "day_index": 1,
    "stop_index": 4,
    "stop": {
      "name": "Miradouro da Senhora do Monte",
      "kind": "attraction",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "13:30",
      "end": "15:00"
    }
  },
  {
    "day_index": 1,
    "stop_index": 5,
    "stop": {
      "name": "Cervejaria Ramiro",
      "kind": "meal",
      "card": {
        "provider": "GOOGLE_MAPS",
        "name": "Cervejaria Ramiro",
        "location": {
          "lat": 38.711,
          "lng": -9.141,
          "crs": "WGS84"
        },
        "category": "restaurant",
        "photos": [
          "https://cdn.example.com/verify_meal_dinner_1.jpg"
        ],
        "sources": [
          {
            "provider": "GOOGLE_MAPS",
            "native_id": "verify_meal_dinner_1",
            "deeplinks": {}
          }
        ]
      },
      "deeplinks": {}
    },
    "slot": {
      "start": "18:30",
      "end": "20:00"
    }
  },
  {
    "day_index": 2,
    "stop_index": 0,
    "stop": {
      "name": "Hills Hotel Lisbon",
      "kind": "stay",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "07:00",
      "end": "07:30"
    }
  },
  {
    "day_index": 2,
    "stop_index": 1,
    "stop": {
      "name": "Castelo de São Jorge",
      "kind": "attraction",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "08:00",
      "end": "09:30"
    }
  },
  {
    "day_index": 2,
    "stop_index": 2,
    "stop": {
      "name": "Miradouro da Senhora do Monte",
      "kind": "attraction",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "10:00",
      "end": "11:30"
    }
  },
  {
    "day_index": 2,
    "stop_index": 3,
    "stop": {
      "name": "A Brasileira",
      "kind": "meal",
      "card": {
        "provider": "GOOGLE_MAPS",
        "name": "A Brasileira",
        "location": {
          "lat": 38.712,
          "lng": -9.142000000000001,
          "crs": "WGS84"
        },
        "category": "restaurant",
        "photos": [
          "https://cdn.example.com/verify_meal_lunch_2.jpg"
        ],
        "sources": [
          {
            "provider": "GOOGLE_MAPS",
            "native_id": "verify_meal_lunch_2",
            "deeplinks": {}
          }
        ]
      },
      "deeplinks": {}
    },
    "slot": {
      "start": "12:00",
      "end": "13:00"
    }
  },
  {
    "day_index": 2,
    "stop_index": 4,
    "stop": {
      "name": "LX Factory",
      "kind": "attraction",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "13:30",
      "end": "15:00"
    }
  },
  {
    "day_index": 2,
    "stop_index": 5,
    "stop": {
      "name": "Prado Restaurant",
      "kind": "meal",
      "card": {
        "provider": "GOOGLE_MAPS",
        "name": "Prado Restaurant",
        "location": {
          "lat": 38.713,
          "lng": -9.143,
          "crs": "WGS84"
        },
        "category": "restaurant",
        "photos": [
          "https://cdn.example.com/verify_meal_dinner_3.jpg"
        ],
        "sources": [
          {
            "provider": "GOOGLE_MAPS",
            "native_id": "verify_meal_dinner_3",
            "deeplinks": {}
          }
        ]
      },
      "deeplinks": {}
    },
    "slot": {
      "start": "18:30",
      "end": "20:00"
    }
  },
  {
    "day_index": 3,
    "stop_index": 0,
    "stop": {
      "name": "Hills Hotel Lisbon",
      "kind": "stay",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "07:00",
      "end": "07:30"
    }
  },
  {
    "day_index": 3,
    "stop_index": 1,
    "stop": {
      "name": "LX Factory",
      "kind": "attraction",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "08:00",
      "end": "09:30"
    }
  },
  {
    "day_index": 3,
    "stop_index": 2,
    "stop": {
      "name": "Torre de Belém",
      "kind": "attraction",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "10:00",
      "end": "11:30"
    }
  },
  {
    "day_index": 3,
    "stop_index": 3,
    "stop": {
      "name": "Time Out Market Lisboa",
      "kind": "meal",
      "card": {
        "provider": "GOOGLE_MAPS",
        "name": "Time Out Market Lisboa",
        "location": {
          "lat": 38.714,
          "lng": -9.144,
          "crs": "WGS84"
        },
        "category": "restaurant",
        "photos": [
          "https://cdn.example.com/verify_meal_lunch_4.jpg"
        ],
        "sources": [
          {
            "provider": "GOOGLE_MAPS",
            "native_id": "verify_meal_lunch_4",
            "deeplinks": {}
          }
        ]
      },
      "deeplinks": {}
    },
    "slot": {
      "start": "12:00",
      "end": "13:00"
    }
  },
  {
    "day_index": 3,
    "stop_index": 4,
    "stop": {
      "name": "Mosteiro dos Jerónimos",
      "kind": "attraction",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "13:30",
      "end": "15:00"
    }
  },
  {
    "day_index": 3,
    "stop_index": 5,
    "stop": {
      "name": "Cervejaria Ramiro",
      "kind": "meal",
      "card": {
        "provider": "GOOGLE_MAPS",
        "name": "Cervejaria Ramiro",
        "location": {
          "lat": 38.715,
          "lng": -9.145000000000001,
          "crs": "WGS84"
        },
        "category": "restaurant",
        "photos": [
          "https://cdn.example.com/verify_meal_dinner_5.jpg"
        ],
        "sources": [
          {
            "provider": "GOOGLE_MAPS",
            "native_id": "verify_meal_dinner_5",
            "deeplinks": {}
          }
        ]
      },
      "deeplinks": {}
    },
    "slot": {
      "start": "18:30",
      "end": "20:00"
    }
  },
  {
    "day_index": 4,
    "stop_index": 0,
    "stop": {
      "name": "Hills Hotel Lisbon",
      "kind": "stay",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "07:00",
      "end": "07:30"
    }
  },
  {
    "day_index": 4,
    "stop_index": 1,
    "stop": {
      "name": "Torre de Belém",
      "kind": "attraction",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "08:00",
      "end": "09:30"
    }
  },
  {
    "day_index": 4,
    "stop_index": 2,
    "stop": {
      "name": "Mosteiro dos Jerónimos",
      "kind": "attraction",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "10:00",
      "end": "11:30"
    }
  },
  {
    "day_index": 4,
    "stop_index": 3,
    "stop": {
      "name": "A Brasileira",
      "kind": "meal",
      "card": {
        "provider": "GOOGLE_MAPS",
        "name": "A Brasileira",
        "location": {
          "lat": 38.716,
          "lng": -9.146,
          "crs": "WGS84"
        },
        "category": "restaurant",
        "photos": [
          "https://cdn.example.com/verify_meal_lunch_6.jpg"
        ],
        "sources": [
          {
            "provider": "GOOGLE_MAPS",
            "native_id": "verify_meal_lunch_6",
            "deeplinks": {}
          }
        ]
      },
      "deeplinks": {}
    },
    "slot": {
      "start": "12:00",
      "end": "13:00"
    }
  },
  {
    "day_index": 4,
    "stop_index": 4,
    "stop": {
      "name": "Castelo de São Jorge",
      "kind": "attraction",
      "card": null,
      "deeplinks": {}
    },
    "slot": {
      "start": "13:30",
      "end": "15:00"
    }
  },
  {
    "day_index": 4,
    "stop_index": 5,
    "stop": {
      "name": "Prado Restaurant",
      "kind": "meal",
      "card": {
        "provider": "GOOGLE_MAPS",
        "name": "Prado Restaurant",
        "location": {
          "lat": 38.717,
          "lng": -9.147,
          "crs": "WGS84"
        },
        "category": "restaurant",
        "photos": [
          "https://cdn.example.com/verify_meal_dinner_7.jpg"
        ],
        "sources": [
          {
            "provider": "GOOGLE_MAPS",
            "native_id": "verify_meal_dinner_7",
            "deeplinks": {}
          }
        ]
      },
      "deeplinks": {}
    },
    "slot": {
      "start": "18:30",
      "end": "20:00"
    }
  }
]
```

## Registry (Lisbon)

- Before: **211** rows
- After: **211** rows
- Sample names: Torre de Belém, Castelo de São Jorge, Museum of Illusions Lisbon, Mosteiro dos Jerónimos, LX Factory, Miradouro da Senhora do Monte, Museum of Lisbon – Pimenta Palace, Lisbon Story Centre…

## Status / timing

- `trip_id`: `cmtqmimkf00024ery56vd92xo`
- `revision`: 30
- `status`: `ready`
- timing: `{
  "intake_s": 0.01,
  "total_s": 0.04,
  "origin_s": 0,
  "skeleton_s": 0,
  "fill_s": 0.02,
  "tips_s": 0
}`

## HTML review

Open [`poc-true-agent-verification.html`](./poc-true-agent-verification.html) in a browser.

## How to re-run

```bash
cd places-agent
npx tsx --env-file=.env.local scripts/verify-poc-true-agent.ts
```

Does not start the HTTP server. Does not call Google Maps.
