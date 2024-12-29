# Simulation-of-a-Mechatronic-Machine
Hydraulically driven crane
Custom hydraulic actuator source code:
component custom_HA
% This is .ssc file template for defining a custom simscape component block
% Nodes are connected to other blocks using the foundation domains.
% domainless inputs and outputs are Simscape signal type.

% Variables include units in them, use '1' if variable has no units,
% value(var,'unit') can be used to strip units from variable and
% {var, 'unit'} to define units for an unitless variable.

nodes
    % Define connection nodes and their name:location in block
    % foundation.hydraulic.hydraulic is for hydraulic connection
    
    % for hydraulic connection on the right side:
    % node = foundation.hydraulic.hydraulic; % name:right
    A = foundation.isothermal_liquid.isothermal_liquid; % A:right
    
    % for rotational mechanical connection:
    % foundation.mechanical.rotational.rotational 
    
    % for translational mechanical connection:
    % foundation.mechanical.translational.translational     
end

inputs
    % Define inputs to block with name and location
    % input = {start value, 'unit'}; % name:location
    x = {0, 'm'}; % x:left
    xdot = {0, 'm/s'}; % xd:left
end

outputs
    % Define outputs from block with name and location
    % output = {start value, 'unit'}; % name:location
    f = {0, 'N'}; % F:left
end

parameters
    % Define the parameters and default values, these can be modified by the user
    % in the component dialog opened by double clicking the block
    % comment after the definition functions as the parameter description in the block
    
    % param = {default value, 'unit'}; % description
    A1 = {pi*(0.05/2)^2, 'm^2'}; % Area on piston side
    V0 = {8e-4, 'm^3'}; % Dead Volume
    Be = {900e6, 'Pa'}; % Effective Bulk Modulus
    n = {0.9, '1'}; % Efficiency of the cylinder
    rho = {850,'kg/m^3'} %Fluid Density
end

variables
    % Inner variables for the block, start value can be modified by user
    % The variable priority can be set to none (default), low or high if needed
    % priority determines if the solver should try to set the value at beginning of simulation

    % var = {{start value, 'unit'}, priority = priority.none }; % description
    % var = {start value, 'unit'}; % description
    pA = {10e5, 'Pa'}; % Pressure on cylinder side
    mdotA = {0, 'kg/s'}; % mass Flow Rate
    
end

variables (Access=private)
    % Inner variables for the block, these cannot be modified by the user
end

branches
    % Define flow of the through variables across the block nodes
    % designate variable qA to present the flow from port A to nowhere (no outlet port)
    
    % through_var: node1.var -> node2.var;
    % use -> *; if there is no second node where the flow goes 
    mdotA: A.mdot -> *;
end
equations
    % Define equations of the block here, equations are evaluated at once 
    % and therefore don't need to be arranged
    
    % let-in-end statement used to define parameters for equations
    let 
        % These are evaluated for use in the latter part
        % if-else-end can be used this way or the usual matlab way
        % else statement is mandatory!
        
        % subvar = if condition, value1 else value2 end;
        VA = if (A1*x) <= V0 , V0 else A1*x end;
    in
        % define the equation where subvar is used here
        % differential variable derivatives can be used with var.der notation
        % '==' defines equal statements
        
        %var == (var2.der * subvar)/const; 
        mdotA == rho*(((pA.der*VA)/Be) + A1*xdot);
    end 
    
    % you can define other equations here algebraic or differential,
    % equation and variable count should match.
    pA == A.p;
    f == A1*pA - xdot/{1,'m/s'}*abs(A1*pA) * (1-n);

    
end
end

Custom 3-way directional valve source code:

component custom_DV
% This is .ssc file template for defining a custom simscape component block
% Nodes are connected to other blocks using the foundation domains.
% domainless inputs and outputs are Simscape signal type.

% Variables include units in them, use '1' if variable has no units,
% value(var,'unit') can be used to strip units from variable and
% {var, 'unit'} to define units for an unitless variable.

nodes
    % Define connection nodes and their name:location in block
    % foundation.hydraulic.hydraulic is for hydraulic connection
    
    % for hydraulic connection on the right side:
    % node = foundation.hydraulic.hydraulic; % name:right
    P = foundation.isothermal_liquid.isothermal_liquid; % P:left
    T = foundation.isothermal_liquid.isothermal_liquid; % T:left
    A = foundation.isothermal_liquid.isothermal_liquid; % A:right
    
    % for rotational mechanical connection:
    % foundation.mechanical.rotational.rotational 
    
    % for translational mechanical connection:
    % foundation.mechanical.translational.translational     
end

inputs
    % Define inputs to block with name and location
    % input = {start value, 'unit'}; % name:location
    Uin = {0, 'V'}; % Uin:left
end

outputs
    % Define outputs from block with name and location
    % output = {start value, 'unit'}; % name:location
    U = {0, 'V'}; % U:right
end

parameters
    % Define the parameters and default values, these can be modified by the user
    % in the component dialog opened by double clicking the block
    % comment after the definition functions as the parameter description in the block
    
    % param = {default value, 'unit'}; % description
    Cv = {2.67261242e-8, 'm^3/(s*V*Pa^0.5)'}; % semi-emperical flow-rate constant
    tau = {1e-4, 's'}; % time constant
    rho = {850, 'kg/m^3'}; % fluid Density
end

variables
    % Inner variables for the block, start value can be modified by user
    % The variable priority can be set to none (default), low or high if needed
    % priority determines if the solver should try to set the value at beginning of simulation

    % var = {{start value, 'unit'}, priority = priority.none }; % description
    % var = {start value, 'unit'}; % description
    p_PA = {0, 'Pa'}; % pP - pA
    p_AT = {0, 'Pa'}; % pA - pT
    mdot_PA = {0, 'kg/s'}; % mass Flow rate from P to A
    mdot_AT = {0, 'kg/s'}; % mass Flow rate from A to T

end

variables (Access=private)
    % Inner variables for the block, these cannot be modified by the user
end

branches
    % Define flow of the through variables across the block nodes
    % designate variable qA to present the flow from port A to nowhere (no outlet port)
    
    % through_var: node1.var -> node2.var;
    % use -> *; if there is no second node where the flow goes 
    mdot_PA: P.mdot -> A.mdot
    mdot_AT: T.mdot -> A.mdot
end

equations
    % Define equations of the block here, equations are evaluated at once 
    % and therefore don't need to be arranged
    
    % let-in-end statement used to define parameters for equations
    let 
        % These are evaluated for use in the latter part
        % if-else-end can be used this way or the usual matlab way
        % else statement is mandatory!
        
        % subvar = if condition, value1 else value2 end;
    in
        % define the equation where subvar is used here
        % differential variable derivatives can be used with var.der notation
        % '==' defines equal statements
        
        %var == (var2.der * subvar)/const; 
    end 
    
    % you can define other equations here algebraic or differential,
    % equation and variable count should match.
    U.der == (Uin - U)/ tau;
    p_PA == P.p - A.p;
    p_AT == A.p - T.p;

    if U > 0
        mdot_PA == rho*(Cv*U*sign(p_PA)*sqrt(abs(p_PA)));
        mdot_AT == 0;
    elseif U < 0
        mdot_PA == 0;
        mdot_AT == rho*(Cv*U*sign(p_AT)*sqrt(abs(p_AT)));
    else
        mdot_PA == 0;
        mdot_AT == 0;
    end

    
end
end
