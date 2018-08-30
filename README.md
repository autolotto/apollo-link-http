# Fork of @apollo-link-http

### Why Fork?

There is at this time a bug in apollo-link-http.

When connecting to another Apollo Server using apollo-link-http and receiving an error, there are a number of possible cases. 

- You receive a network error e.g. something like a HTTP 500
- You receive a GraphQL Error `{ errors: [ ... ] }`
- You receive BOTH; a GraphQL Error with a network error

The final case, where you will receive both a graphql error and network error is not properly handled by apollo-link-http. 

An example of this kind of unique error is a parse error which Apollo uniquely treats as a network error + graphql error.  We were able to simulate this by for example using an invalid Date format which couldn't be parsed.   The parser is pre-GraphQL so it seems to be considered a network error. 

HOWEVER The code to handle this case is present but its inside an `if` block with an error in the condition that prevents the code from ever running.  Perhaps this used to work with a legacy version of Apollo and has broken. 

This means that in this event of a combined network/graphql Error, the GraphQL Error is not emitted to observables.  Since this doesn't happen, other links in the chain cannot handle the error. 

This has the final consequence of basically breaking schema-stitching.  When a stitched schema returns such an error, you WANT the GraphQL Error to be stitched into your errors section even if there is also a network error (non-200 code)  but it is not.  This leads to much confusion / swallowed errors, when one service among many that you are stitching retuns such an error. 

Hopefully this is eventually solved, but for now, this is a 1 line patch/fork that I will submit to Apollo's team.





