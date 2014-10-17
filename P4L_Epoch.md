PlotManagement<br>
By Zupa and Rosska85<br><br>

Installation
============

**STEP 1 (Copying Files)**

First, unpack your mission file.<br>
Now copy the "plotManagement" folder to your mission directory.

**STEP 2 (Modifying description.ext)**<br>
At the bottom, add

	#include "plotManagement\defines.hpp"
	#include "plotManagement\plotManagement.hpp"

**Note** If you already have a custom defines.hpp don't use ours.

**STEP 3 (Modifying compiles.sqf)**<br>
Add

	/*Plot*/
	PlotGetFriends 		= compile preprocessFileLineNumbers "plotManagement\plotGetFriends.sqf";
	PlotNearbyHumans 	= compile preprocessFileLineNumbers "plotManagement\plotNearbyHumans.sqf";
	PlotAddFriend 		= compile preprocessFileLineNumbers "plotManagement\plotAddFriend.sqf";
	PlotRemoveFriend 	= compile preprocessFileLineNumbers "plotManagement\plotRemoveFriend.sqf";
	/*Plot End*/
	
**STEP 4 (Modifying variables.sqf)**<br>
After
	
	//Player self-action handles
	dayz_resetSelfActions = {
	
Add

	s_player_plotManagement = -1;
	
**Note** If you don't already have a custom variables.sqf then create one and call if after the standard variables.sqf in your mission init.sqf file. <br>
Add this to it
	//Player self-action handles
	dayz_resetSelfActions = {
		s_player_plotManagement = -1;
	};
	call dayz_resetSelfActions;
	

**STEP 5 (Modifying fn_selfActions.sqf)**<br>
**5 A**<br>
Find 

	if (s_player_maintain_area < 0) then {
	
Directly above that, add

	if (s_player_plotManagement < 0) then {
		_adminList = ["0152"]; // Add admins here if you admins to able to manage all plotpoles
		_owner = _cursorTarget getVariable ["ownerPUID","0"];
		_friends = _cursorTarget getVariable ["plotfriends", []];
		_fuid = [];
		{
		_friendUID = _x select 0;
		_fuid = _fuid + [_friendUID];
		} forEach _friends;
		_allowed = [_owner];    
		_allowed = [_owner] + _adminList + _fuid;
		if((getPlayerUID player) in _allowed)then{            
		s_player_plotManagement = player addAction ["<t color='#0059FF'>Manage Plot</t>", "plotManagement\initPlotManagement.sqf", [], 5, false];
		};
	};

**5 B**<br>
Find
	
	} else {
    	player removeAction s_player_maintain_area;
    	s_player_maintain_area = -1;
    	player removeAction s_player_maintain_area_preview;
    	s_player_maintain_area_preview = -1;
	};
	
Replace that with

	} else {
		player removeAction s_player_plotManagement;
		s_player_plotManagement = -1;
    	player removeAction s_player_maintain_area;
    	s_player_maintain_area = -1;
    	player removeAction s_player_maintain_area_preview;
    	s_player_maintain_area_preview = -1;
	};

**5 C**<br>
Find
	
	//Allow owners to delete modulars
        if(_isModular && (_playerUID == _ownerID)) then {
                if(_hasToolbox && "ItemCrowbar" in _itemsPlayer) then {
                        _player_deleteBuild = true;
                };
        };
	//Allow owners to delete modular doors without locks
        if(_isModularDoor && (_playerUID == _ownerID)) then {
                if(_hasToolbox && "ItemCrowbar" in _itemsPlayer) then {
                        _player_deleteBuild = true;
                };      
        };  

Replace that with

	///Allow owners to delete modulars
    if(_isModular) then {
            if(_hasToolbox && "ItemCrowbar" in _itemsPlayer) then {
				_findNearestPoles = nearestObjects[player, ["Plastic_Pole_EP1_DZ"], DZE_PlotPole select 0];
				_IsNearPlot = count (_findNearestPoles);
				_fuid  = [];
				_allowed = [];
				if(_IsNearPlot > 0)then{
					_thePlot = _findNearestPoles select 0;
					_owner =  _thePlot getVariable ["ownerPUID","010"];
					_friends = _thePlot getVariable ["plotfriends", []];
					{
					  _friendUID = _x select 0;
					  _fuid  =  _fuid  + [_friendUID];
					} forEach _friends;
					_allowed = [_owner];    
					_allowed = [_owner] +  _fuid;	
					if ( _playerUID in _allowed && _ownerID in _allowed ) then {  
						_player_deleteBuild = true;
					};					
				}else{
					if(_ownerID == _playerUID)then{
						_player_deleteBuild = true;
					};
				};						                  
            };
    };
	//Allow owners to delete modular doors without locks
    if(_isModularDoor) then {
            if(_hasToolbox && "ItemCrowbar" in _itemsPlayer) then {			
				_findNearestPoles = nearestObjects[player, ["Plastic_Pole_EP1_DZ"], DZE_PlotPole select 0];
				_IsNearPlot = count (_findNearestPoles);
				_fuid  = [];
				_allowed = [];
				if(_IsNearPlot > 0)then{
					_thePlot = _findNearestPoles select 0;
					_owner =  _thePlot getVariable ["ownerPUID","010"];
					_friends = _thePlot getVariable ["plotfriends", []];
					{
					  _friendUID = _x select 0;
					  _fuid  =  _fuid  + [_friendUID];
					} forEach _friends;
					_allowed = [_owner];    
					_allowed = [_owner] +  _fuid;	
					if ( _playerUID in _allowed && _ownerID in _allowed) then {
						_player_deleteBuild = true;
					};					
				}else{
					if(_ownerID == _playerUID)then{
						_player_deleteBuild = true;
					};
				};								
            };      
    };

**5 D**<br>
Find

	} else {
		//Engineering
	
After that, add

	player removeAction s_player_plotManagement;
	s_player_plotManagement = -1;
	
**STEP 6 (Modifying remove.sqf)**<br>

No more changes needed in this file. RESTORE TO THE NORMAL REMOVES.SQF
	
**STEP 7 (Modifying player_build.sqf, player_upgrade.sqf, and player_buildingDowngrade.sqf)**<br>
ALL THREE OF THESE FILES NEED THE SAME EDIT, MAKE SURE YOU DO ALL FILES!!!!<br>
Find
	
	_friendlies		= player getVariable ["friendlyTo",[]];
	// check if friendly to owner
	if(_ownerID in _friendlies) then {
		_canBuildOnPlot = true;
	};

Replace that with

	_friendlies = _nearestPole getVariable ["plotfriends",[]];
	_fuid  = [];
	{
		  _friendUID = _x select 0;
		  _fuid  =  _fuid  + [_friendUID];
	} forEach _friendlies;
	_builder  = getPlayerUID player;
	// check if friendly to owner
	if(_builder in _fuid) then {
		_canBuildOnPlot = true;
	};	

**STEP 8 This one is in your dayz_server.pbo (Modifying server_monitor.sqf)**<br>
**8 A**<br>
Find
	
	_object setVariable ["ObjectID", _idKey, true];
	
After that, add

	if (typeOf (_object) == "Plastic_Pole_EP1_DZ") then {
	_object setVariable ["plotfriends", _intentory, true];
	};
	
**8 B**<br>
Find

	if (count _intentory > 0) then {
	
Replace that with

	if ((count _intentory > 0) && !(typeOf( _object) == "Plastic_Pole_EP1_DZ")) then {
	
**STEP 9 Again, this is in your dayz_server.pbo (Modifying server_updateObject.sqf)**<br>
Find

	_inventory = [
	getWeaponCargo _object,
	getMagazineCargo _object,
	getBackpackCargo _object
	];
	
Replace that with

	if (typeOf (_object) == "Plastic_Pole_EP1_DZ") then{
		_inventory = _object getVariable ["plotfriends", []]; //We're replacing the inventory with UIDs for this item
	} else {
		_inventory = [
		getWeaponCargo _object,
		getMagazineCargo _object,
		getBackpackCargo _object
		];
	};
	
Installation complete<br>

**Infistar Antihack**<br>
If you're running Infistar Antihack, add this to the dialogs array;

	711194
	
And this to the '_cMenu =' section

	"PlotManagement"