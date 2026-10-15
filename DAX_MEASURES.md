# DAX measures - technical reference

This reference records the formulas confirmed during project
documentation, plus the agreed calculation patterns for department
measures. Department-specific column names not supplied verbatim are
deliberately not reconstructed.

## Attendance and averages

``` dax
Total A&E attendances = SUM(Activity[Total Attendances])

Average daily attendance =
DIVIDE(
    [Total A&E attendances],
    SUMX(VALUES(DimDate[Period]), DAY(EOMONTH(DimDate[Period], 0)))
)

Average monthly attendance =
AVERAGEX(VALUES(DimDate[Year]), [Total A&E attendances])
```

The monthly-average measure is used with a calendar-month filter to
average the selected month's total across years.

## Year-on-year analysis

``` dax
previous year attendance =
CALCULATE([Total A&E attendances], DATEADD(DimDate[Period], -1, YEAR))

YoY attendance change =
[Total A&E attendances] - [previous year attendance]

YoY attendance change % =
DIVIDE([YoY attendance change], [previous year attendance])

Validated Previous Year Attendance =
VAR CurrentYear = SELECTEDVALUE(DimDate[Year])
RETURN
    IF(CurrentYear >= 2012, [previous year attendance], BLANK())

validated YoY growth % =
VAR CurrentYear = SELECTEDVALUE(DimDate[Year])
RETURN
    IF(
        ISBLANK(CurrentYear) || CurrentYear < 2012,
        BLANK(),
        [YoY attendance change %]
    )
```

The annual chart/table filter also excludes partial 2026.

## Four-hour performance

``` dax
Calculated 4 hour performance % =
DIVIDE(
    SUM(Performance[Total Attendances < 4 hours]),
    SUM(Performance[Total Attendances < 4 hours])
        + SUM(Performance[Total Attendances > 4 hours])
)

Type 1 4 Hour Performance % =
DIVIDE(
    SUM(Performance[Type 1 Departments - Major A&E (<  4 hours)]),
    SUM(Performance[Type 1 Departments - Major A&E (<  4 hours)])
        + SUM(Performance[Type 1 Departments - Major A&E  (> 4 hours)])
)
```

`Type 2 4 Hour Performance %` and `Type 3 4 Hour Performance %` use the
same under-four-hours divided by under-plus-over-four-hours structure
with their respective department columns.

## Department attendance totals and shares

``` dax
Type 1 attendances = SUM(Activity[Type 1 Departments - Major A&E])

Type 1 Share % =
DIVIDE([Type 1 Attendances], [Total A&E Attendances])

Type 2 Share % =
DIVIDE([Type 2 Attendances], [Total A&E Attendances])

Type 3 Share % =
DIVIDE([Type 3 Attendances], [Total A&E Attendances])
```

`Type 2 Attendances` and `Type 3 Attendances` are `SUM()` measures over
their corresponding department columns in `Activity`.

## Booked attendances

``` dax
Total booked attendances = SUM(Booking[Booked_Total attendances])

Booked share of A&E % =
DIVIDE([Total booked attendances], [Total A&E attendances])
```

## Validation

-   Total A&E attendances: **382,411,747**.
-   Weighted four-hour performance: **84.7%**.
-   Average daily attendance: **65,091.36**.
-   2025 vs 2024 attendance growth: **1.35%**.

Reporting limitations and annual date restrictions are described in the
main README.
