## Comparison of temporal discretization schemes in OpenFOAM

Various temporal schemes are examined on a 1-D structured mesh, where a
passive scalar (T) is convected along the Cartesian x-axis using a
uniform flow velocity of (1, 0, 0).  Executing:

./Allrun (choose appropriate OpenFOAM version)
OpenFOAM v6 tested! 
OpenFOAM v1912 not verified!

The Allrun script will update the system/fvSchemes file to use each scheme listed in the file system/schemesToTest and perform the test using the scalarTransportFoam solver.

A test file for each run is generated that can be used to compare the predicted
scalar profile against the exact solution.

### Domain

Domain extends [0,1] in a Cartesian XY plane.
A sinusoidal field *(T(y) = sin(2*pi*y))* is applied on the left (x = 0) at time 0, and convected toward the right (along x axis).

**ScalarField T**
	
	internalField   uniform 0;

	boundaryField
	{
	    top
	    {
	        type            fixedValue;
	        value           uniform 0;
	    }

	    bottom
	    {
	        type            fixedValue;
	        value           uniform 0;
	    }

	    left
	    {
	        type            codedFixedValue;
	        value           uniform 1;
	        name            sinY;
	        code            #{
	            const vector axis(0, 1, 0);
	            scalarField v(this->patch().Cf() & axis);
	            operator==(sin(constant::mathematical::twoPi*v));
	        #};
	    }

	    right
	    {
	        type            zeroGradient;
	    }

	    frontAndBack
	    {
	        type            empty;
	    }
	}

**Velocity U**
	
	internalField   uniform (1 0 0);

	boundaryField
	{
	    top
	    {
	        type            zeroGradient;
	    }

	    bottom
	    {
	        type            fixedValue;
	        value           $internalField;
	    }

	    left
	    {
	        type            fixedValue;
	        value           $internalField;
	    }

	    right
	    {
	        type            zeroGradient;
	    }

	    frontAndBack
	    {
	        type            empty;
	    }
	}


Results are extracted y = 0.25 (i.e. pi/2 when scaled by 2 pi) which shall be analytically 1. We expect dissipative nature from Euler implicit (1st order) scheme and oscillatory behavior from backward (2nd order upwind implicit) scheme. 
A modification is proposed to OpenFOAM CrankNicolson scheme. Visit the **schemeComparison.ipynb** file to visualize scheme comparison