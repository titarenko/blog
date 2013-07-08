---
layout: post
comments: true
title: "BDD with SpecFlow and WatiN"
categories: .NET Testing
---

I've long heard about BDD, and recently viewed lectures from [SaaS class](http://saas-class.org) really interested me not only in Ruby (with such great frameworks as Rails and Cucumber), but also in practical use of this methodology in .NET, which is my main platform at the moment. So, I decided to try writing small web app using BDD approach assisted by SpecFlow tool for definition of features and latter execution of test based on them, and WatiN as tool which would help to grab content from browser to make assertions on expected results.

Here is the feature file:

```gherkin
Feature: Viewing of rates
	In order to view available conversion rates
	As a user
	I want to see current rates as well as their changes history

Scenario: View current rates
	Given I have following exchange rates in the system
		| From | To  | Rate |
		| USD  | UAH | 8.03 |
		| RUR  | UAH | 0.27 |
	When I am on home page
	Then I should see following rows
		| V1  | V2  | V3     |
		| USD | UAH | 8.0300 |
		| RUR | UAH | 0.2700 |
```

Implementation of most steps is pretty straightforward and not listed here, except "Then" step. So, below is its initial implementation.

```csharp
[Then(@"I should see following rows")]
public void ThenIShouldSeeFollowingRows(Table table)
{
    foreach (var row in table.Rows)
    {
        Assert.That(
            Browser.Table("exchange-rates").TableRows.Any(
                exchangeRate => row.Values.All(
                    text => exchangeRate.TableCell(Find.ByText(text)).Exists)),
            string.Format("Page contains following row: {0}.", row.ToReadableString()));
    }
}
```

Being WatiN newbie, I'd made a step on a rake and trial to execute scenario led to StackOverflowException in WatiN.Core, so after googling a bit I decided to parse table first and only then make assertions. Said - done: updated implementation is below.

```
csharp[Then(@"I should see following rows")]
public void ThenIShouldSeeFollowingRows(Table table)
{
    var actualTable = Browser
        .Table("exchange-rates")
        .TableRows
        .Where(r => r.TableCells.Any())
        .Select(r => r.TableCells.Select(c => c.Text.Trim()).ToArray())
        .ToArray();

    foreach (var row in table.Rows)
    {
        Assert.That(
            actualTable.Any(r => row.All(v => r.Contains(v.Value))),
            string.Format("Page contains following row: {0}.", row.ToReadableString()));
    }
}
```

Please note that Trim() call on text of cell: it's important since text can be, for example, with trailing space (as it was in my case).

So, finally I can get to actual implementation of this feature and then, running the above scenario, obtain long-awaited result.

```
ViewCurrentRates : Passed
Given I have following exchange rates in the system
  --- table step argument ---
  | From | To  | Rate |
  | USD  | UAH | 8.03 |
  | RUR  | UAH | 0.27 |
-> done: DataPreparationSteps.GivenIHaveFollowingExchangeRatesInTheSystem(<table>) (7,9s)
When I am on home page
-> done: NavigationSteps.WhenIAmOnPage("home") (5,1s)
Then I should see following rows
  --- table step argument ---
  | V1  | V2  | V3     |
  | USD | UAH | 8.0300 |
  | RUR | UAH | 0.2700 |
-> done: VerificationSteps.ThenIShouldSeeFollowingRows(<table>) (0,4s)
```

Overall impressions: even if it adds some difficulties (especially at the beginning), I think it's great idea to practice BDD, and as for tooling: both SpecFlow and WatiN are nice and intuitive, and it cost quite low effort to get familiar with them (at least in this certain case).
