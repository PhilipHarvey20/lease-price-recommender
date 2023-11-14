
<img src="https://raw.githubusercontent.com/PhilipHarvey20/lease-price-recommender/master/Images/Open_Woods_Logo_OW.png" width="200">


# Outdoor Property Lease Calculator - 
A lease price calculator for landowners who lease their property for hunting, fishing, hiking, camping, metal detecting, and more!


# Why pay to rent land?

Although public land and waterways are free to use, the quality of hunting, fishing, and metal-detecting is often very poor due to overcrowding, restrictive regulations, and poor land management. Leasing private land for outdoor recreation is one solution to this. 

In Central Florida, for example, there are 29 Wildlife Management Areas (WMA) within a 1-hour drive of Orlando, but a first-year hunter or new Florida resident will only be able to hunt 11/29 of them. Why? Because the other 18 WMAs require a quota permit (lottery system) where only hunters whose names are drawn from a lottery are allowed to hunt there. Other hobbies such as metal-detecting and farming are completely prohibited on public land.


![Image-WMAs_Requiring_Quota_Permit_Cent_FL](https://raw.githubusercontent.com/PhilipHarvey20/lease-price-recommender/master/Images/Image-WMAs_Requiring_Quota_Permit_Cent_FL.png
)








# Current State of Leasing Private Land
Most websites for renting recreational land only provide the landowner a place to upload pictures of their property and the option to list a phone number. These sites do not help with pricing or creating lease contracts. Renters, as a result, find inflexible listings offering  expensive annual leases. The goal of the Outdoor Property Lease Calculator is to help landowners calculate an appropriate lease price for any lease duration and property size to encourage landowners to offer more flexible contracts at multiple price points.

# Step 1 - Calculating Annual Lease Price (Hunting)
Initially, I would like to calculate lease prices only for hunting since  leasing private land for hunting is already common. Unfortunately, I was not able to find enough data on hunting lease prices to fit a model. Instead, I will start with a small infographic I found on deerpros.com showing the average price per acre of hunting land in each state. I will use these values in a variable called: ```base_state_price```. 

![Image Alt Text](https://raw.githubusercontent.com/PhilipHarvey20/lease-price-recommender/master/Images/Avg_Cost_Per_Acre_by_State.png) (source: https://deerpros.com/hunting-lease/hunting-lease-price-guide/)

# Annual Price Formula

Now that we have a base acreage price per state. To calculate an annual lease price, I will use the following formula:

```annual_price = base_state_price * acreage```

For example a 125-acre property in Alabama priced at $10/acre would cost ```$1,250``` a year:

```annual_price = base_state_price * acreage```

```125 acres * $10 = $1250```















# Step 2 - 1st Day Lease Price
Now that we know how much to charge for a 1-year lease, let's next calculate how much to charge for a 1-day lease. The price of a 1-day lease will simply be ```5%``` of the ```annual_price```. 

```first_day_price = annual_price * .05```

The 125-acre Alabama property from earlier would cost ```$62.50``` to rent for one day. 

```$1250 * .05` = $62.50```



# Step 3 - Calculate Final Price

Next, we need to calculate how much a property should cost for all of the lease durations between 1 and 365 days. Before we do that, remember that most State's hunting seasons are between 90-150 days. For example, in Alabama where the deer hunting season is 120 days, the price of a 120-day lease should cost about as much as the `annual_acre_price` since 120 days in Alabama is essentially a full "year" of hunting.

To achieve that, I will use a function where `lease_price` increases steeply at lower `lease_durations` and gradually stabilizes around day 90-150 (depending on the State). We can use an exponential decay function with a horizontal reflection over the x-axis. Basically a "flipped" exponential decay curve 

# Step 3.1 - Final Price Formula

To calculate the final price, I will use the following formula:

`final_price = first_day_price + additional_days_price `

# Step 3.2 - Calc additional_days_price

We already have the `first_day_price`, but to calculate the `additional_days_price`, we will iterate backwards from day 365 by 1 for each day in`lease_duration` + 1. We will multiply each day by the `annual_acre_price` and a State-specific `decay_rate`. The difference between each day's final value is summed and added to the `additional_days_price` variable. 


  # Calculate Additional days price
```python additional_days = 1 # Start counting lease days after day 1
        days_in_year = [day for day in range(1,367)] # ::-1 gets reverse of list 
        for day in (days_in_year[:self.lease_duration]): 
            additional_days += 1
            current_day_price = self.annual_acre_price  * (np.exp(self.decay_rate * day)) #
            current_day_price_plus_day = self.annual_acre_price  * (np.exp(self.decay_rate * (day + 1))) 
            addl_day_price = abs(current_day_price_plus_day - current_day_price)
            self.additional_days_price += addl_day_price # Total price of all additional lease days after day 1
        # Calculate Final Price
        self.final_price = self.first_day_price + self.additional_days_price
```

# Step 3.3 - Setting Custom Decay Rates

I mentioned earlier that each State's hunting season length can vary, so we will want to ensure that a State's `final_price` is 90% of the `annual_acre_price ` when the `lease duration` is equal to the length of the hunting season. For example, the hunting season in Alabama is 120 days, so if someone rents a property in Alabama for 120 days, they should pay 90% of the 'full-year' price or `annual_acre_price`. To achieve this, for Alabama we will use a decay rate of `-0.017`, but each state will have its own unique decay rate to achieve this 90% target. To better understand this, let's look at two plots: the cost of a 100-acre property in Maryland vs. a 100-acre property in Florida. 


![Image Alt Text](https://raw.githubusercontent.com/PhilipHarvey20/lease-price-recommender/master/Images/Cost_of_100_Acre_Lease_MD_vs_FL.1.png)

Since Florida's hunting season is longer than Maryland's, it will take longer for the price to reach 90% of the annual price.

# Step 4 - Conclusion and Next Steps

The prices are looking great! The next step will be calculating lease prices for other activities such as fishing, metal detecting, etc. 
