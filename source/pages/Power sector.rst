############
Power sector
############

* Mainly drawn from the International Electricity Market Module (IEMM in the WEPS+ framework) project for Energy Information Administration (EIA – USA)
* Vintaged buildup of existing stock from Platts database
* Renewable electricity modeled at a country-level, regardless of the model regional aggregation, for Wind, Solar, Hydro and Biomass
* Electricity demand load curves: sector load shape archetypes, based on 8760 grid-level data and assumptions based on sector shares
* .. raw:: html

    <a href="https://www.eia.gov/analysis/handbook/pdf/weps2021_electricity.pdf" target="_blank"><b>Further details</a></b>

In IEMM, the bound and constraint facilities have been used to construct two sets of limitations to the penetration of variable renewable (VRE) generation. The need for these limits arises because VRE has inherently granular temporal and spatial dependencies. However, detailed the time and space resolution of a global model such as IEMM, it will always be insufficient to capture these dependencies fully. Specifically, we wish to address these two challenges:
	* Limiting the use of country-level wind and solar penetration in large, possibly multi-country regions
	* Limiting the ability of variable resources to meet average loads in model time slices.

These have been addressed as follows.

Controlling use of country-level VRE potential
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
IEMM contains country-level wind and solar potentials, but models demand at a regional level. In multi-country regions, as well as in large countries, resources may not be located near loads, and transmission capacity may not exist from resources to load. We do not want the model satisfying an unrealistic portion of regional load with renewables that may be located in a small and potentially distant portion of the region.
To provide a generic structure to control the penetration of these technologies, IEMM allows each country to produce a maximum level of generation from wind-plus-solar before it has to incur a grid extension cost. To allow generation beyond that limit, three levels of "grid extension" investment, notionally representing short distance, medium distance, and long distance transmission, have been provided each with assumed bounds and investment costs.
The bounds for the initial maximum generation and the three levels of grid extension are initially driven by projected country load, based upon a logistic curve that is fitted to minimize the sum of squared errors of per capita consumption from historical values and population projections.

For each country n, a user constraint requires:
    .. math::
        Annual Generation from VRE_n ≤ δ×ProjectedLoad_n+(SDGrid_n+ MDGrid_n+ LDGrid_n)×8.76

The maximum capacity of each of the three additional "grids" is bounded as a share of ProjectedLoad:
    .. math::
        SDGrid_n ≤ α ×PMHG_n⁄8.76

        MDGrid_n ≤ β ×PMHG_n⁄8.76

        LDGrid_n ≤ γ ×PMHG_n⁄8.76

Values δ, α, β, and γ  and the investment cost of each grid are under user control and can be changed in the model instance generator.
Note that these constraints are implemented at the country level, regardless of how countries are aggregated into model regions. They are intended to prevent, for example, Iceland's abundant wind resources from powering all of Europe's load, without appropriate transmission investment.
Similar constraints may be added to prevent over-utilization of hydro resources in countries with large potential, without corresponding transmission investment.

Bounding time-sliced generation from VRE sources
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
As described in Section 2, model time slices are designed to capture two important types of temporal variation in load and generation: seasonal and time-of-day. They do not capture day-to-day variation within seasons. Rather they average over such variation. When penetrations of VRE sources become large relative to total load, such averaging may run the risk of implicitly assuming free storage between days or even hours within the same time slice.
To avoid such an outcome, user constraints have been implemented that limit the maximum share of total electricity production in each region and time slice from VRE sources. Output from storage technologies is included in the denominator of the constraint, resulting in a requirement for storage investment and operation when VRE penetrations become very high. The default maximum VRE share is set at 65%. This value can be changed in the model instance generator template.

Constraints on capacity rate of change
======================================
IEMM contains constraints on capacity contraction and expansion intended to represent real-world costs and inertias that limit capacity rate of change. These are implemented as decay constraints and build rate growth constraints and cost steps.

Decay constraints
^^^^^^^^^^^^^^^^^
Non-economic capacity is often retained in the real world due to local must-run considerations, institutional practices, and other factors. In IEMM, decay constraints have been imposed that limit the rate of decrease of coal, gas, and oil capacity. The constraints include a maximum annual percentage rate of decline, as well as a representative unit size that can be retired above and beyond the annual percentage rate, in order to allow the capacity to go to zero.

Build rate constraints/costs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
IEMM contains a set of three-step cost curves, similar to the NEMS Electricity Market Module short-term elasticity mechanism, intended to represent the costs incurred when rapid expansions in the capacity of a particular technology cause shortages of key inputs and/or skilled labor. These constraints have been implemented for renewable technologies, whose economics are rapidly changing, to more realistically limit their rate of expansion as their costs fall. The constraints include a cap on investment in the first projection year, based on recent regional additions, and a maximum annual capacity investment growth rate in subsequent periods.

The current constraint permits additions to capacity to grow at an annual maximum rate of 15% without incurring additional cost. Further steps of 70% and another 15% are available at an extra cost to the model.



