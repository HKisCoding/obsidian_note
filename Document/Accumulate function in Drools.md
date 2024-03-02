## Scenario: 
- Calculate the accumulate in the LHS on the go with backdate of 1/2/3 months
- Check how Drools keeping fact -> effect on the performance

## Accumulate function

``` drl
ResultPattern( fieldconstraint* )
from accumulate ( SourcePattern( fieldconstraint* )
                  init( code )
                  action( code )
                  reverse( code )
                  result( code ) )
```

- Operate on sets of data
- Iterate over the facts that match the SourcePattern executing the action code block for each of the facts and finally executing the result code block.

