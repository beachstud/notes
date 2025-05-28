Notes
=====

**Purpose of this file**
  
  Some notes on issues that were a PITA to resolve and that might be helpful for others.

----



.. contents:: **Table of Contents**


MacOS
-----

These applications are essential to make MacOS usable:
  
  * `Mos <https://mos.caldis.me>`_: normal smooth scrolling with the mouse. No silly erratic scrolling or in the wrong direction.
  * `Little Snitch <https://obdev.at/products/littlesnitch/index.html>`_: firewall.
  * `ForkLift <https://binarynights.com>`_: file explorer.
  * `Strongbox <https://strongboxsafe.com>`_: password manager.
  * Extension in Safari: AdGuard, Dark Reader.
  * `TinkerTool <https://www.bresink.com/osx/TinkerTool.html>`_: has some interesting features.
  
  
Computer freezes with i915
--------------------------

**Issue**
  
  System freezes either once every couple of days or more frequently if disk io is high. Completely unresponsive, requires power cycle.
  
**What the problem was:**
  
  i915 intel embedded GPU on the Intel Xeon E3-1245 v6

**What solved it for me:**
  
  Disable low power states, for instance by adding the following to your NixOS configuration:
  
  .. code:: nix
    
    boot.kernelModules = [ "i915" ];
    boot.kernelParams = [
      "i915.enable_guc=2"
      "i915.enable_psr=0"
      "i915.enable_dc=0"
      "i915.enable_fbc=0"
    ];


NixOS flake set-up for multiple architectures
---------------------------------------------

**Issue**
  
  Some of my devices/hosts are x86_64-linux and some are aarch64-linux. Most online solutions to this problem are too cumbersome or old/incorrect, such as importing nixpkgs.

**Solution:**
  
  Use a helper function in your ``flake.nix``:
  
  .. code:: nix
  
    {
      description = "System configurations";
    
      inputs = {
        nixpkgs.url = "github:NixOS/nixpkgs/nixos-25.05";
      };
    
      outputs = inputs@{
        self,
        nixpkgs,
        ...
      }: let
      inherit (self) outputs;
    
      # Helper function to create a nixos system configuration
      # Usage:
      #   Default x86_64:  mkSystem { host = "hostname"; };
      #   Custom system:   mkSystem { host = "hostname"; system = "aarch64-linux"; };
      mkSystem = { host, system ? "x86_64-linux", hostId, timeZone }:
        nixpkgs.lib.nixosSystem {
          system = system;
          specialArgs = {
            inherit inputs outputs;
            # Some additional variables that I pass along, not essential.
            host = {
              hostName = host;
              hostId = hostId;
              timeZone = timeZone;
            };
          };
          modules = [
            # Import your configuration.nix for each host here as usual.
            ./hosts/${host}.nix
          ];
        };
      in {
        nixosConfigurations = {
          hostA = mkSystem { host = "hostA"; hostId = "abchostA"; timeZone = "Australia/Sydney"; };
          hostB = mkSystem { host = "hostB"; hostId = "abchostB"; timeZone = "Australia/Sydney"; system = "aarch64-linux"; };
        };
      };
    }
