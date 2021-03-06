## Sample golang code

```go
package main

import (
	"log"
	"time"
)

// type FeatureType string
type PlanFeature struct {
	FeatureID int
	Type      string
	Value     string
}

type Plan struct {
	ID        int
	ValidFrom time.Time
	ValidTill time.Time
	Cost      float64
	Name      string
	// The relationship should be inversed.
	// FeatureType FeatureType // Size, Country, etc.
	// The feature that it supersedes.
	ParentID int
}

func main() {
	/*
		Price of Coffee
		| id | country | size | currency | valid_from | valid_till | parent_id | price |
		| id | malaysia | small | MYR | 2019-01-01 | 9999-12-31 | - | 7 |
		| id | malaysia | medium | MYR | 2019-01-01 | 9999-12-31 | - | 8 |
		| id | malaysia | large | MYR | 2019-01-01 | 2019-01-02 | - | 9 |
		| id | malaysia | large | MYR | 2019-01-02 | 9999-12-31 | - | 10 |
	*/

	plans := []Plan{
		{
			ID:        1,
			Cost:      7,
			ValidTill: time.Date(9999, 12, 31, 0, 0, 0, 0, &time.Location{}),
		},
		{
			ID:        2,
			Cost:      9,
			ValidTill: time.Date(9999, 12, 31, 0, 0, 0, 0, &time.Location{}),
		},
		{
			ID:        3,
			Cost:      10,
			ValidTill: time.Now(),
		},
		{
			ID:        4,
			Cost:      11,
			ValidFrom: time.Now(),
			ValidTill: time.Date(9999, 12, 31, 0, 0, 0, 0, &time.Location{}),
			ParentID:  3,
		},
	}

	var planFeatures []PlanFeature
	planFeatures = []PlanFeature{
		{1, "size", "small"},
		{1, "country", "malaysia"},
	}
	// Select valid features.
	for _, p := range plans {
		valid := struct {
			start, end bool
		}{
			time.Now().Equal(p.ValidFrom) || time.Now().After(p.ValidFrom),
			time.Now().Before(p.ValidTill),
		}
		if valid.start && valid.end {
			log.Println(p)
		}
	}
	// Select features for the feature type 1
	for _, planFeature := range planFeatures {
		if planFeature.FeatureID == 1 {
			log.Println(planFeature)
		}
	}
}
```

Note: The design above only considers one value. What if the pricing is affected by multiple pricing?

## References:

- https://dba.stackexchange.com/questions/216317/pricing-table-mysql-table-design-i-need-the-database-design-for-the-the-plans
- https://softwareengineering.stackexchange.com/questions/307214/designing-pricing-table-rdbms-agnostic
- https://stackoverflow.com/questions/14546539/database-design-for-pricing-overview
