with alltransaction as
(
select block_time, 
    success, gas_price/10^9 as gas_price,
    hash, gas_used,
    max(gas_price),
    (gas_price*gas_used)/10^18 as eth_tx_used
from Ethereum.transactions
where "from" = '\x1a2a960F61E9546b23F32eb09C252010dc5Bf8Ff'
group by block_time,success,gas_price,hash,gas_used
)
/**select block_time, count(*) over (order by block_time desc) as cum_tx from alltransaction**/
/**SELECT date_trunc('week',block_time), count(*) over (order by block_time desc) as cum_tx from alltransaction**/
/** order by gas_price desc**/
select date_trunc('minute',block_time),
/**price**/
eth_tx_used,
sum(eth_tx_used*price) over (order by date_trunc('minute',block_time) desc) as cum_usd_Paid_for_TX
/**hash**/
from alltransaction as a
left join (select minute,price from prices.usd where symbol='WETH') as prices on date_trunc('minute',block_time)=minute 
