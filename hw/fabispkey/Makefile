#
# Makefile for generating gerbers and pngs.
# Tested on Advanced Circuits barebones, should work for 33 each, 
# gerbmerge, and Advanced Circuits. Pngs used for the fab modules 
# and the modella. 
#
# Copyright (C) 2011 Andy Bardagjy. 
#
# This makefile is free software; you can redistribute it and/or
# modify it under the terms of the GNU Lesser General Public
# License as published by the Free Software Foundation; either
# version 2.1 of the License, or (at your option) any later version.
# 
# This makefile is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# Lesser General Public License for more details.
# 
# You should have received a copy of the GNU Lesser General Public
# License along with this library; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA
#
# Andy Bardagjy
# June 21, 2011
#
# TODO
# * Interesting bug, if there are no holes on the board, the script fails
# 	because no drd file is generated, so... I touch an empty drd file 
# 	so the build doesn't fail. This is kinda a kluge. 
# *	If there's no dimension layer, the build fails.
# * Noticed that when gerbv has a boarder of 0% (with flag -B0) sometimes
#   dimension is cut off (especially if linewidth of 0). This is fixed for 
#	document export, but remains as is for fab png export (should be fixed?)
#

PROJECT?= fabisp

WHITE?= \#FFFFFF
BLACK?= \#000000

all: gerbers fab parts docs

# Component Side Gerber
$(PROJECT).cmp: $(PROJECT).brd
	eagle -X -dGERBER_RS274X -o$(PROJECT).cmp $(PROJECT).brd Top Pads Vias

# Solder Side Gerber
$(PROJECT).sol: $(PROJECT).brd
	eagle -X -dGERBER_RS274X -o$(PROJECT).sol $(PROJECT).brd Bottom Pads Vias

# Top Soldermask Gerber
$(PROJECT).stc: $(PROJECT).brd
	eagle -X -dGERBER_RS274X -o$(PROJECT).stc $(PROJECT).brd tStop

# Bottom Soldermask Gerber
$(PROJECT).sts: $(PROJECT).brd
	eagle -X -dGERBER_RS274X -o$(PROJECT).sts $(PROJECT).brd bStop

# Top Silkscreen Gerber
$(PROJECT).plc: $(PROJECT).brd
	eagle -X -dGERBER_RS274X -o$(PROJECT).plc $(PROJECT).brd Dimension tPlace tNames tValues

# Bottom Silkscreen Gerber
$(PROJECT).pls: $(PROJECT).brd
	eagle -X -dGERBER_RS274X -o$(PROJECT).pls $(PROJECT).brd Dimension bPlace bNames bValues

# Drills Xcellon
$(PROJECT).drd: $(PROJECT).brd
	# FIXME This is the drd hack (see top of file)
	rm -f $(PROJECT).drd
	touch $(PROJECT).drd
	eagle -X -dEXCELLON -o$(PROJECT).drd $(PROJECT).brd Drills Holes

# Board Outline Gerber
$(PROJECT).bor: $(PROJECT).brd
	eagle -X -dGERBER_RS274X -o$(PROJECT).bor $(PROJECT).brd Dimension

# Top Copper PNG
# So you *can* do this with eagle commands, but the holes don't work out right.
# Gotta use gerbv to do this
#$(PROJECT)_cmp.png: $(PROJECT).brd
#	eagle -C "ratsnest; display none Top Pads Vias; export image $(PROJECT)_cmp.png monochrome 600; quit;" $(PROJECT).brd
#	convert $(PROJECT)_cmp.png -negate $(PROJECT)_cmp.png
$(PROJECT)_cmp.png: gerbers
	gerbv gerbers/$(PROJECT).cmp gerbers/$(PROJECT).bor -b$(BLACK) -f$(WHITE) -f$(WHITE) -B0 -D600 -o $(PROJECT)_cmp.png -a -xpng
	convert $(PROJECT)_cmp.png -colorspace gray +dither -colors 2 -normalize $(PROJECT)_cmp.png
	convert $(PROJECT)_cmp.png -mattecolor white -frame 25x25 $(PROJECT)_cmp.png
	convert $(PROJECT)_cmp.png -strip $(PROJECT)_cmp.png
	convert $(PROJECT)_cmp.png -units PixelsPerInch -density 600 -colorspace sRGB -type TrueColor PNG32:$(PROJECT)_cmp.png

# Bottom Copper PNG
# So you *can* do this with eagle commands, but the holes don't work out right.
# Gotta use gerbv to do this
$(PROJECT)_sol.png: gerbers
	gerbv gerbers/$(PROJECT).sol gerbers/$(PROJECT).bor -b$(BLACK) -f$(WHITE) -f$(BLACK) -B0 -D600 -o $(PROJECT)_sol.png -a -xpng
	convert $(PROJECT)_sol.png -colorspace gray +dither -colors 2 -normalize $(PROJECT)_sol.png
	convert $(PROJECT)_sol.png -mattecolor white -frame 25x25 $(PROJECT)_sol.png
	convert $(PROJECT)_sol.png -strip $(PROJECT)_sol.png
	convert $(PROJECT)_sol.png -units PixelsPerInch -density 600 -colorspace sRGB -type TrueColor PNG32:$(PROJECT)_sol.png

# Drills PNG
# So you *can* do this with eagle commands, but the holes don't work out right.
# Gotta use gerbv to do this
#$(PROJECT)_drd.png: $(PROJECT).brd
#	eagle -C "ratsnest; display none Drills Holes Dimension; export image $(PROJECT)_drd.png monochrome 600; quit;" $(PROJECT).brd
$(PROJECT)_drd.png: gerbers
	gerbv gerbers/$(PROJECT).drd gerbers/$(PROJECT).bor -b$(WHITE) -f$(BLACK) -f$(BLACK) -B0 -D600 -o $(PROJECT)_drd.png -a -xpng
	convert $(PROJECT)_drd.png -colorspace gray +dither -colors 2 -normalize $(PROJECT)_drd.png
	convert $(PROJECT)_drd.png -mattecolor black -frame 25x25 $(PROJECT)_drd.png
	convert $(PROJECT)_drd.png -strip $(PROJECT)_drd.png
	convert $(PROJECT)_drd.png -units PixelsPerInch -density 600 -colorspace sRGB -type TrueColor PNG32:$(PROJECT)_drd.png

gerbers: $(PROJECT).cmp $(PROJECT).sol $(PROJECT).stc $(PROJECT).sts $(PROJECT).plc $(PROJECT).pls $(PROJECT).drd $(PROJECT).bor
	mkdir -p gerbers
	mv $(PROJECT).{bor,cmp,drd,dri,gpi,sol,stc,sts,plc,pls} gerbers/
	zip gerbers/$(PROJECT)_advanced.zip gerbers/*.{bor,drd,cmp,sol,stc,sts,plc,pls}
	zip gerbers/$(PROJECT)_barebones.zip gerbers/*.{bor,drd,cmp,sol}

# fab: $(PROJECT)_cmp.png $(PROJECT)_drd.png $(PROJECT)_sol.png
fab: $(PROJECT)_cmp.png $(PROJECT)_drd.png
	mkdir -p fab
	# mv $(PROJECT){_cmp.png,_sol.png,_drd.png} fab/
	mv $(PROJECT){_cmp.png,_drd.png} fab/

view: gerbers
	#gerbv gerbers/$(PROJECT).{bor,drd,stc,plc,cmp,sts,pls,sol} &
	gerbv gerbers/$(PROJECT).{bor,drd,cmp,sol} &

parts: $(PROJECT).sch
	eagle -C "export partlist $(PROJECT).txt; quit;" $(PROJECT).sch
	eagle_parts $(PROJECT).csv $(PROJECT).txt
	mkdir parts
	mv $(PROJECT).{csv,txt} parts

$(PROJECT)_sch.png: $(PROJECT).sch
	eagle -C "export image $(PROJECT)_sch.png 300; quit;" $(PROJECT).sch
	convert $(PROJECT)_sch.png -mattecolor white -frame 25x25 $(PROJECT)_sch.png

$(PROJECT)_tbrd.png: $(PROJECT).brd
	eagle -X -dGERBER_RS274X -o$(PROJECT).tpop $(PROJECT).brd Dimension tPlace tNames Document Reference tDocu
	gerbv $(PROJECT).tpop -b$(WHITE) -f$(BLACK) -B5 -D600 -o $(PROJECT)_tbrd.png -a -xpng
	convert $(PROJECT)_tbrd.png -mattecolor white -frame 25x25 $(PROJECT)_tbrd.png

$(PROJECT)_bbrd.png: $(PROJECT).brd
	eagle -X -dGERBER_RS274X -o$(PROJECT).bpop $(PROJECT).brd Dimension bPlace bNames Document Reference bDocu
	gerbv $(PROJECT).bpop -b$(WHITE) -f$(BLACK) -B5 -D600 -o $(PROJECT)_bbrd.png -a -xpng
	convert $(PROJECT)_bbrd.png -mattecolor white -frame 25x25 $(PROJECT)_bbrd.png

docs: $(PROJECT)_sch.png $(PROJECT)_tbrd.png $(PROJECT)_bbrd.png
	mkdir docs
	mv $(PROJECT)_{sch,tbrd,bbrd}.png docs
	rm $(PROJECT).{tpop,bpop,gpi}

clean:
	rm -rf $(PROJECT).{csv,txt,bor,cmp,drd,dri,gpi,sol,zip,png,path,sts,stc,pls,plc,tpop,bpop}
	rm -rf $(PROJECT)_{sch.png,tbrd.png,bbrd.png}
	rm -rf *#*
	rm -rf gerbers fab parts docs
