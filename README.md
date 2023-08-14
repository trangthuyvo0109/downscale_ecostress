# downscale_ecostress
This scrip shows the instructions how to use the Python script to downscale the LST from ECOSTRESS LST images to get the urban element LSTs 

Step 1: Define the values of independent variables (ek(i,j) * fk(i,j)): land cover fraction list for each pixel (for example, a single pixel has 10% of tree cover, 20% of bare soil cover, etc). To get the fraction of each land cover class, you can use 'Zonal Statistics' in python (e.g., rasterstats). ek(i,j) is the default emissivity of each land cover class, here, we are specify a fixed values for each land cover class. (righ-hand side of Eq. 1 in the paper) 
        
coeff_df_element = coeff_df.groupby('index')['value_fraction'].apply(lambda x: x.values).to_numpy()
    # Coefficiene matrix values: fraction i * emis i 
 coeff_df_element = list(coeff_df_grp["value_fraction"].apply(lambda x: x.values) * coeff_df_grp["value_emis"].apply(lambda x: x.values))

Step 2: Define the values of dependent variables (LST(i,j)^4 * e(i,j)) from the ECOSTRESS values (left-hand Eq. 1 in the paper)
 
        # LST^4:
        lst4 = np.array(list(map(lambda x:pow(x,4),
                                     indep_df.groupby('index')["value_lst"].apply(lambda x: x.values[0]).to_numpy())))
        # emissivity:
        emis_sum = indep_df.groupby('index')["value_emis_sum"].apply(lambda x: x.values[0]).to_numpy()
            
        # Dependent values: Element-wise multiplication 
        dep_df_element = [a * b for a, b in zip(lst4,emis_sum)]

Step 3: as a result, for each pixel (70x70 m), we will have one set of equation with eight unknown LSTk (LST of each land cover class)
Step 4: to solve the eight unknown variables, we will need at least 8 equations correspondingly. For this reason, we apply 5x5 pixels moving window after conducting sensitivity test to define which moving kernels is most appropriate (please follow the documentation for more detail about the sensitivity test of moving window). 
To apply the MLR method for these set of equations, we use following code: 
   
from scipy.optimize import lsq_linear

 res = lsq_linear(coeff_df_element,np.array(dep_df_element).reshape(len(dep_df_element),)
                        , bounds=bounds)

Note that, bounds in the equation is the constrained bounding was set based on the maximum (upper constraint) and minimum (lower constraint) LSTs in the domain for each ECOSTRESS image. 
