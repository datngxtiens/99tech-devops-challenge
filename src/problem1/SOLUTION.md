Provide your CLI command here:
<pre>jq -r 'select(.symbol=="TSLA" and .side=="sell") | .order_id' ./transaction-log.txt | xargs -I{} curl -s "https://example.com/api/{}" >> ./output.txt</pre>

### Detailed Explanation

<b>Objective:</b> Submit a HTTP GET request to https://example.com/api/:order_id with Order IDs that are selling TSLA, writing to output to ./output.txt

<b>Initiative:</b> To make HTTP GET request to the provided endpoints, we first need to get all of the Order IDs from the transaction log file that meet two conditions:
* Transaction of the Order ID must come from the "side" of "sell"
* Transaction of the Order ID must have "symbol" of "TSLA"

<b>Resolution:</b> With the above initatives, I come up with the resolution of a single-line command as above, in which we first extract all the qualified Order IDs using the ```jq``` command. This command parses the JSON data from ```transaction-log.txt``` file, gets the Order ID that meets the condition and passes it to the next command via pipe. The next command used is ```xargs```. It uses flag ```-I{}``` to replace the ```{}``` in the ```curl``` command with the Order ID passed as argument from the previous pipe. The ```curl``` command will in turn run to make the GET request to the endpoint and write output to the ```output.txt``` file