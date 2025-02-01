# controller

## create
```
self.editor_menu = undefined;
self.update_texture = false;

if (global.PE){ 
    return}
global.PE = {
    --//CHANGE THIS TO TRUE IF YOU DON'T WANT THE EXTRA PROMPT ABOUT P FOR PALETTE EDITOR ON SAVE POINTS
    vanillaSaveTextbox : false,

    --//CHANGE THIS TO TRUE IF YOU DON'T WANT THE EDITOR WINDOW TO MOVE
    turnOffReposition : false,

    --//CHANGE THIS TO TRUE IF YOU HAVE ANY OTHER MOD THAT REPLACES OPLAYER!!!
    turnOffPlayerReplace : false,

    --//EXPERIMENTAL!!! MIGHT BREAK STUFF!!!
    --//CHANGE THIS TO TRUE IF YOU WANT HAIR ON MAYA
    forceHair : false,

    noAmeli : false,
    noMaya : false,
    modPath : "mods/rmml/palette_editor/",
    paletteH : 56,
    palette_data : [],
    hair_number : [8, 1, 4],
    splayer_palette : 0,
    shue : 0,
    scolor_rect : 0,
    salpha : 0,
    part_names_grid : 0,
    splayer_head : 0,
    spalette_bg : 0,
    splayer_palette_ref : 0
}
global.maya_mode = 0;
global.ameli_mode_ = 0;

let splayer_palette_fern = if (file_exists(global.PE.modPath + "saved_palettes/palette_fern.png"))
    {
        global.rmml.log("Loading Fern palette...");
        sprite_add(global.PE.modPath + "saved_palettes/palette_fern.png", 0, false, false, 0, 0);
    }
    else if (file_exists("Fern_custom/palette.png"))
    {
        global.rmml.log("Loaded palette from Fern_custom");
        sprite_add("Fern_custom/palette.png", 0, false, false, 0, 0);
    }
    else
    {
        global.rmml.log("No palettes found for Fern! Loading backup pallette");
        sprite_add(global.PE.modPath + "backup_palette.png", 0, false, false, 0, 0);
    }
    
let splayer_palette_maya = if (file_exists(global.PE.modPath + "saved_palettes/palette_maya.png"))
    {
        global.rmml.log("Loading Maya palette...");
        sprite_add(global.PE.modPath + "saved_palettes/palette_maya.png", 0, false, false, 0, 0);
    }
    else if (file_exists("mods/rmml/maya_palette/palette.png"))
    {
        global.rmml.log("No palettes found for Maya! Loading backup pallette");
        sprite_add("mods/rmml/maya_palette/palette.png", 0, false, false, 0, 0);
    }
    else
    {
        global.rmml.log("maya_palette mod not found! No palettes for Maya today(");
        global.PE.noMaya = true;
        splayer_palette_fern;
    }
let splayer_palette_ameli = if (file_exists(global.PE.modPath + "saved_palettes/palette_ameli.png"))
    {
        global.rmml.log("Loading Ameli palette...");
        sprite_add(global.PE.modPath + "saved_palettes/palette_ameli.png", 0, false, false, 0, 0);
    }
    else if (file_exists("mods/rmml/ameli_palette/palette.png"))
    {
        global.rmml.log("No palettes found for Ameli! Loading backup pallette");
        sprite_add("mods/rmml/ameli_palette/palette.png", 0, false, false, 0, 0);
    }
    else
    {
        global.rmml.log("ameli_palette mod not found! No palettes for Ameli today(");
        global.PE.noAmeli = true;
        splayer_palette_fern;
    }

global.PE.splayer_palette = [splayer_palette_fern, splayer_palette_maya, splayer_palette_ameli];
global.PE.paletteH = sprite_get_height(global.PE.splayer_palette[0]);
global.palette_texture = sprite_get_texture(global.PE.splayer_palette[0], 0);

if (!global.player_use_shader)
{
    global.player_use_shader = true;
    global.player_palette_shader_ = shd_palette;
    shader_replace_simple_set_hook(global.player_palette_shader_);
    global.main_shader_palette_pointer = shader_get_sampler_index(global.player_palette_shader_, "palette");
    let main_shader_col_num_pointer = shader_get_uniform(global.player_palette_shader_, "col_num");
    let main_shader_pal_num_pointer = shader_get_uniform(global.player_palette_shader_, "pal_num");
    let main_shader_pal_index_pointer = shader_get_uniform(global.player_palette_shader_, "pal_index");
    let main_shader_uvs_pointer = shader_get_uniform(global.player_palette_shader_, "palette_uvs");

    shader_set_uniform_f(main_shader_col_num_pointer, global.PE.paletteH);
    shader_set_uniform_f(main_shader_pal_num_pointer, 2);
    shader_set_uniform_f(main_shader_pal_index_pointer, 1);
    shader_set_uniform_f_array(main_shader_uvs_pointer, [0,0, 1,1]);

    shader_replace_simple_reset_hook();
}

let genPaletteData = fun (palette_sprite)
{
    let palette_data = array_create(global.PE.paletteH);
    if (palette_sprite == undefined){
        return palette_data;}
    let surf = surface_create(1, global.PE.paletteH);
    surface_set_target(surf);
    draw_clear_alpha(c_black, 0);
    gpu_set_blendmode_ext( bm_one, bm_zero );
    draw_sprite(palette_sprite, 0, -1, 0);
    gpu_set_blendmode(bm_normal);
    let buff = buffer_create(global.PE.paletteH * 4, buffer_fixed, 1);
    buffer_get_surface(buff, surf, 0);
    let i = 0;
    buffer_seek(buff, buffer_seek_start, 0);
    while (i < global.PE.paletteH)
    {
        palette_data[i] = buffer_read(buff, buffer_u32);
        i += 1;
    }
    surface_reset_target();
    surface_free(surf);
    buffer_delete(buff);
    return palette_data;
}
global.PE.palette_data = [genPaletteData(splayer_palette_fern), genPaletteData(splayer_palette_maya), genPaletteData(splayer_palette_ameli)];

if (file_exists(global.PE.modPath + "saved_palettes/hair_data.ini"))
{
    ini_open(global.PE.modPath + "saved_palettes/hair_data.ini");
    global.PE.hair_number[0] = ini_read_real("hair_data", "length", 8);
    global.PE.hair_number[1] = ini_read_real("hair_data", "maya_length", 1);
    global.PE.hair_number[2] = ini_read_real("hair_data", "ameli_length", 4);
    global.hair_start_col_ = ini_read_real("hair_data", "start_col", 4279836483);
    global.hair_end_col_ = ini_read_real("hair_data", "end_col", 4281478686);
    ini_close();
}
else
{
    global.hair_start_col_ = 4279836483;
    global.hair_end_col_ = 4281478686;
    ini_open(global.PE.modPath + "saved_palettes/hair_data.ini")
    ini_write_real("hair_data", "length", global.PE.hair_number[0]);
    ini_write_real("hair_data", "maya_length", global.PE.hair_number[1]);
    ini_write_real("hair_data", "ameli_length", global.PE.hair_number[2]);
    ini_write_real("hair_data", "start_col", global.hair_start_col_);
    ini_write_real("hair_data", "end_col", global.hair_end_col_);
    ini_close();
}
global.hair_number_ = global.PE.hair_number[0];

global.PE.shue = sprite_add(global.PE.modPath + "hue.png", 0, false, false, 0, 0);
global.PE.scolor_rect = sprite_add(global.PE.modPath + "color_rect.png", 0, false, false, 0, 0);
global.PE.salpha = sprite_add(global.PE.modPath + "alpha.png", 0, false, false, 0, 0);
global.PE.part_names_grid = load_csv(global.PE.modPath + "part_names.csv");
global.PE.splayer_head = sprite_add(global.PE.modPath + "head.png", 2, false, false, 0, 0);
global.PE.spalette_bg = sprite_add(global.PE.modPath + "palette_bg.png", 0, false, false, 0, 0);
let splayer_palette_ref_fern = sprite_add(global.PE.modPath + "splayer_palette_ref_fern.png", 0, false, false, 0, 0);
let splayer_palette_ref_maya = sprite_add(global.PE.modPath + "splayer_palette_ref_maya.png", 0, false, false, 0, 0);
let splayer_palette_ref_ameli = sprite_add(global.PE.modPath + "splayer_palette_ref_ameli.png", 0, false, false, 0, 0);
global.PE.splayer_palette_ref = [splayer_palette_ref_fern, splayer_palette_ref_maya, splayer_palette_ref_ameli];

if (!global.PE.turnOffPlayerReplace){
    alarm_set(0, 1);}
```

## alarm_0
```
let loadSprites = fun(path)
{
    let f = file_find_first(path + "*.png", 0)
    let i = 0;
    while (f != "")
    {
        let name = string_split(f, ".png")[0];
        let index = asset_get_index(name);
        let subimage_number = sprite_get_number(index);
        let xoffset = sprite_get_xoffset(index);
        let yoffset = sprite_get_yoffset(index);
        match (index) {
            case smaya_legs_idle {
                sprite_replace(global.__ameli_idle, path + f, subimage_number, false, false, xoffset, yoffset);
            }
            case smaya_legs_run {
                sprite_replace(global.__ameli_run, path + f, subimage_number, false, false, xoffset, yoffset);
            }
            case splayer_maya_legs_crouching {
                sprite_replace(global.__ameli_crouch, path + f, subimage_number, false, false, xoffset, yoffset);
            }
            else {
                sprite_replace(index, path + f, subimage_number, false, false, xoffset, yoffset);
            }
        }
        sprite_replace(index, path + f, subimage_number, false, false, xoffset, yoffset);
        f = file_find_next();
        i += 1;
    }
    file_find_close();
    return i
}

if (!global.PE.noMaya)
{
    global.rmml.log("Replacing Sprites for Maya...");
    if (loadSprites("mods/rmml/maya_palette/DO_NOT_TOUCH/") > 0)
    {
        global.rmml.log("Sprites Replaced!");
    }
    else 
    {
        global.rmml.log("Replacement Failed! Is everything alright with maya_palette mod?");
    }
    global.__scarf_charged = global.PE.palette_data[1][54];
    global.__scarf_uncharged = global.PE.palette_data[1][55];
}
if (!global.PE.noAmeli)
{
    global.rmml.log("Replacing Sprites for Ameli...");
    if (loadSprites("mods/rmml/ameli_palette/DO_NOT_TOUCH/") > 0)
    {
        global.rmml.log("Sprites Replaced!");
    }
    else 
    {
        global.rmml.log("Replacement Failed! Is everything alright with ameli_palette mod?");
    }
    global.__ameli_hair = global.PE.palette_data[2][55];
}
```

## room_start
```
if (room_get() == rm_menu and !self.update_texture)
{
    self.update_texture = true;
}
else if (self.update_texture)
{
    let mode = global.maya_mode + global.ameli_mode_ * 2;
    global.palette_texture = sprite_get_texture(global.PE.splayer_palette[mode], 0);
    global.hair_number_ = global.PE.hair_number[mode];
    if (global.PE.turnOffPlayerReplace and self.mode != 0)
    {
        shader_replace_simple_set_hook(global.player_palette_shader_);
        shader_set_uniform_f_array(shader_get_uniform(global.player_palette_shader_, "palette_uvs"), [0,0, 0,0]);
        shader_replace_simple_reset_hook();
    }
    self.update_texture = false;
}
```

## step
```
if ((global.PE.noAmeli and global.ameli_mode_) or (global.PE.noMaya and global.maya_mode)){
    return; }

if ((global.maya_mode or global.ameli_mode_))
{
    with (oplayer)
    {
        if (!self.swapped_to_mod and !global.PE.turnOffPlayerReplace)
        {
            instance_change(omod_player, false);
            self.swapped_to_mod = true;
            self.mod_name = global.rmml_current_mod;
        }
        if (!self.hair_init and self.ameli_inited)
        {
            self.hair_number = global.hair_number_;
            self.hair_init = true;
        }
    }
}

if (!instance_exists(osave_point)){ 
    return }

let check = false;
with(osave_point){ 
    check = place_meeting(self.x, self.y, oplayer) and (!self.delay); }

if (keyboard_check_pressed('P'))
{
    if(instance_exists(self.editor_menu)){
        with(self.editor_menu){ 
            instance_destroy(); }}
    --//if near save statue make editor window
    else if (check){ 
        self.editor_menu = instance_create_depth(0, 0, -100, omod_instance); }
}

if (!instance_exists(self.editor_menu)){ 
    return }

check = false; 
with (ogame){
    if (self.draw_map){ 
        check = true; }}
--//if map has been opened destroy editor window
if (check){
    with(self.editor_menu){ 
        instance_destroy(); }}
```

## draw
```
if ((global.PE.noAmeli and global.ameli_mode_) or (global.PE.noMaya and global.maya_mode)){
    return; }

if(!instance_exists(osave_point) or self.vanillaSaveTextbox){ 
    return }

--//draw prompt with open editor hotkey
let save_effect = 0;
with (ogame){
    save_effect = self.save_effect; }

with (osave_point)
{
    if (place_meeting(self.x, self.y, oplayer) and (global.gamestate == 1) and (save_effect <= 0))
    {
        if (self.text_box != self and self.text_box != self.text_box2){
            instance_destroy(self.text_box); }

        let key1 = input_to_visual("down");
        let key2 = input_interact_string();
        if (self.text_box2 == undefined)
        {
            self.text_box2 = create_9slice(self.x, self.y - 64, (key1 + " " + ui_load_entry("save:menu_new", 0) + "\n" + key2 + " " + ui_load_entry("save:menu_new", 1) + "\nP = Edit Palette"), 1);
            self.text_box = self.text_box2;
        }
    }
    else if ((self.text_box2 != undefined))
    {
        instance_destroy(self.text_box2);
        self.text_box2 = undefined;
    }
}
```

# instance

## create
```
self.inv255 = 1 / 255;
self.ox = 370;
self.oy = 16;
self.wh = 64;
self.headW = sprite_get_width(global.PE.splayer_head);
self.headH = sprite_get_height(global.PE.splayer_head);
self.colRectWH = 19;
self.HueH = 5;
self.AlphaH = 5;
self.HairButtonsWH = 8;
self.updateOffsets = fun()
{
    self.offsetW = self.ox + self.wh;
    self.offsetH = self.oy + self.wh;
    self.offsetHueY = self.offsetH + 4;
    self.offsetAlphaY = self.offsetHueY + self.HueH + 2;
    self.offsetPartNameX = self.ox + 1;
    self.offsetPartNameY = self.offsetAlphaY + self.AlphaH + 10;
    self.offsetColRectY = self.offsetPartNameY + 8;
    self.offsetTextY = self.offsetColRectY + self.colRectWH + 5;
    self.offsetHeadX = self.ox + 27;
    self.offsetHeadY = self.offsetColRectY + 2;
    self.offsetHairButtonsX = self.offsetHeadX + self.headW + 2;
    self.offsetHairButtonsY = self.offsetHeadY + self.headH - (self.HairButtonsWH >> 1);
    self.offsetPreviewX = self.ox - 14;
    self.offsetPreviewY = self.offsetTextY + 63;
}
self.updateOffsets();

self.isHovering = false;
self.isHoveringHue = false;
self.isHoveringAlpha = false;
self.isHoveringSatVal = false;
self.isHoveringHairLength = false;
self.isSelectedHue = false;
self.isSelectedAlpha = false;
self.isSelectedSatVal = false;
self.isSelectedPalette = false;
self.isSelectedHair = false;
self.isSelectedHairLength = false;

self.hairLengthButton = 0;
self.timer = 0;
self.pressTimer = 0;
self.pressTimerMax = 3;
self.mode = global.maya_mode + global.ameli_mode_ * 2;
self.hairNumber_ = global.PE.hair_number[self.mode];

self.generateHex = fun()
{
    let hex = "0123456789ABCDEF";
    let col = self.color;
    let i = 3;
    self.hex = "#";
    while (i > 0)
    {
        self.hex += string_copy(hex, (col >> 4 & 15) + 1, 1) + string_copy(hex, (col & 15) + 1, 1);
        col = col >> 8;
        i -= 1;
    }
}

self.selectedPart = 0;
self.updatePicker = fun(col)
{
    self.color = col;
    self.h = colour_get_hue(col);
    self.s = colour_get_saturation(col);
    self.v = colour_get_value(col);
    self.a = (col >> 24) * self.inv255;
    self.HueX = ((self.h * self.inv255)) * (self.wh + 1);
    self.SatValX = (1 - (self.s * self.inv255)) * (self.wh + 1);
    self.SatValY = (1 - (self.v * self.inv255)) * (self.wh + 1);
    self.AlphaX = (self.a) * (self.wh + 1);

    self.R = colour_get_red(col);
    self.G = colour_get_green(col);
    self.B = colour_get_blue(col);
    self.generateHex();
}
self.updatePicker(global.PE.palette_data[self.mode][self.selectedPart]);

self.updateHair = fun(length)
{
    if (length == undefined){ 
        self.hairNumber_ -= self.hairLengthButton; }
    else{ 
        self.hairNumber_ = length; }
    self.hairNumber_ = clamp(self.hairNumber_, 1, 255)
    global.PE.hair_number[self.mode] = floor(self.hairNumber_);
    global.hair_number_ = global.PE.hair_number[self.mode];
    
    let other = self.id;
    with (oplayer)
    {
        self.hair_number = global.PE.hair_number[other.mode];
        let i = self.hair_number;
        self.hair = [];
        while (i)
        {
            array_push(self.hair, struct_gen());
            i -= 1;
        }
    }
}
self.updateHair();

if (global.PE.turnOffPlayerReplace and self.mode != 0)
{
    shader_replace_simple_set_hook(global.player_palette_shader_);
    shader_set_uniform_f_array(shader_get_uniform(global.player_palette_shader_, "palette_uvs"), [0,0, 1,1]);
    shader_replace_simple_reset_hook();
}
```

## step
```
--//move editor window if it's over the player
let px = 0;
with (ocamera)
{
    let xpos = self.xpos;
    with (oplayer){ 
        px = self.x - xpos; }
}

if (px > 315 and !global.PE.turnOffReposition)
{
    self.ox = 18;
    self.updateOffsets();
}

if (px < 127 and !global.PE.turnOffReposition)
{
    self.ox = 370;
    self.updateOffsets();
}

let ctrl = keyboard_check(vk_control)
let str = "";

--//paste currently selected color's hex
if(ctrl and keyboard_check_pressed('V'))
{
    if (clipboard_has_text()){
        str = clipboard_get_text(); }
    if (str != "")
    {
        str = string_upper(string_lettersdigits(string_trim(str)));
        if (string_length(str) == 6)
        {
            let hex = "0123456789ABCDEF";
            let col = 0;
            let i = 6;
            while (i > 0)
            {
                col = col | (((string_pos(string_char_at(str, i), hex) - 1)) | ((string_pos(string_char_at(str, i - 1), hex) - 1) << 4));
                col = col << 8;
                i -= 2;
            }
            col = col >> 8;
            col = col | ((self.a * 255) << 24)
            self.updatePicker(col);
            alarm_set(0, 1);
        }
    }
}

--//copy currently selected color's hex
if(ctrl and keyboard_check_pressed('C'))
{
    clipboard_set_text(self.hex);
}

--//check if cursor is hovering over a ui object
let cx = global.mouse_gui_x_;
let cy = global.mouse_gui_y_;
self.isHovering = point_in_rectangle(cx, cy, self.ox - 19, self.oy - 10, self.offsetW + 10, self.oy + 234);
self.isHoveringHue = point_in_rectangle(cx, cy, self.ox, self.offsetHueY, self.offsetW + 1, self.offsetHueY + self.HueH + 1);
self.isHoveringAlpha = point_in_rectangle(cx, cy, self.ox, self.offsetAlphaY, self.offsetW + 1, self.offsetAlphaY + self.AlphaH + 1);
self.isHoveringSatVal = point_in_rectangle(cx, cy, self.ox, self.oy, self.offsetW + 1, self.offsetH + 1);
self.isHoveringHairLength = point_in_rectangle(cx, cy, self.offsetHairButtonsX, self.offsetHairButtonsY, self.offsetHairButtonsX + (self.HairButtonsWH << 1) + 1, self.offsetHairButtonsY + self.HairButtonsWH + 1);

if (self.isHoveringHairLength){ 
    self.hairLengthButton = (((cx < (self.offsetHairButtonsX + self.HairButtonsWH)) << 1) - 1) * 0.4; }


if(mouse_check_button_pressed(1))
{
    self.isSelectedHue = self.isHoveringHue;
    self.isSelectedAlpha = self.isHoveringAlpha;
    self.isSelectedSatVal = self.isHoveringSatVal;
    self.isSelectedPalette = point_in_rectangle(cx, cy, self.ox - 9, self.oy, self.ox - 5, self.oy + 224);
    self.isSelectedHair = point_in_rectangle(cx, cy, self.offsetHeadX, self.offsetHeadY, self.offsetHeadX + (self.headW >> 1), self.offsetHeadY + self.headH) and (self.mode == 0 or global.PE.forceHair);
    self.isSelectedHairLength = self.isHoveringHairLength and (self.mode != 1 or global.PE.forceHair);

    if (self.isSelectedHairLength)
    {
        self.timer = 0;
        self.pressTimer = self.pressTimerMax;
        self.updateHair(self.hairNumber_ - (self.hairLengthButton * 2.5));
    }
}

if(mouse_check_button(1))
{
    --//making it so you can't shoot while editor window is up
    global.input_skip_ = 1;
    --//fuck you boltcaster!
    if (global.current_weapon_ == 2){
        with(oinput){ 
            self.button_1[0] = 0;}}
    if (!self.isHovering){
        with(oplayer){
            self.shoot_delay = 2;}}

    if(self.isSelectedSatVal)
    {
        self.SatValX = clamp(cx, self.ox, self.offsetW + 1) - self.ox;
        self.SatValY = clamp(cy, self.oy, self.offsetH + 1) - self.oy;
        
        self.s = 255 - floor(((self.SatValX) / (self.wh + 1)) * 255);
        self.v = 255 - floor(((self.SatValY) / (self.wh + 1)) * 255);

        alarm_set(0, 1);
    }

    if(self.isSelectedHue)
    {
        self.HueX = clamp(cx, self.ox, self.offsetW + 1) - self.ox;

        self.h = floor(((self.HueX) / (self.wh + 1)) * 255);

        alarm_set(0, 1);
    }

    if(self.isSelectedAlpha)
    {
        self.AlphaX = clamp(cx, self.ox, self.offsetW + 1) - self.ox;

        self.a = self.AlphaX / (self.wh + 1);

        alarm_set(0, 1);
    }

    if (self.isSelectedPalette)
    {
        self.selectedPart = clamp((cy - self.oy) >> 2, 0, 55);
        
        self.updatePicker(global.PE.palette_data[self.mode][self.selectedPart]);
    }

    if (self.isSelectedHair and (self.mode == 0 or global.PE.forceHair))
    {
        self.selectedPart = clamp(floor((cy - (self.offsetHeadY)) * 0.1), 0, 1) + 56;
        let col = match (self.selectedPart) 
        {
            case 56 {global.hair_start_col_} 
            case 57 {global.hair_end_col_}
            else    {c_black}
        }

        self.updatePicker(col);
    }

    if (self.isSelectedHairLength and (self.mode != 1 or global.PE.forceHair))
    {
        if (self.timer >= 30){
            self.updateHair(); }
        else {
            self.timer += 1}
        self.pressTimer = self.pressTimerMax;
    } 
}
else
{
    self.isSelectedHue = false;
    self.isSelectedAlpha = false;
    self.isSelectedSatVal = false;
    self.isSelectedPalette = false;
    self.isSelectedHair = false;
    self.isSelectedHairLength = false;
}
```

## draw_gui_end
```
if (!instance_exists(oplayer)){
    return; }

--//draw ui background
draw_sprite_stretched(sui_9slice, 0, self.ox - 5, self.oy - 5, self.wh + 11, 234);
draw_sprite_stretched(sui_9slice, 0, self.ox - 14, self.oy - 5, 14, 234);
draw_sprite_stretched(sui_9slice, 0, self.ox - 5, self.offsetAlphaY + self.AlphaH, self.wh + 11, 18);

let SatValX = self.SatValX + self.ox;
let SatValY = self.SatValY + self.oy;
let HueX = self.HueX + self.ox;
let HueY = self.offsetHueY;
let AlphaX = self.AlphaX + self.ox;
let AlphaY = self.offsetAlphaY;
let col = self.color;

draw_set_color(#3f4245);

--//draw saturation and value selector
let col1 = make_colour_hsv(self.h, 255, 255);
let col2 = make_colour_hsv(self.h, 0, 255);
let col3 = make_colour_hsv(self.h, 255, 0);
let col4 = make_colour_hsv(self.h, 0, 0);
draw_rectangle_color(self.ox, self.oy, self.offsetW, self.offsetH, col1, col2, col3, col4, false);
draw_rectangle(self.ox, self.oy, self.offsetW, self.offsetH, true);
draw_sprite_stretched(scursor, 1, SatValX - 1.5, SatValY - 1.5, 3, 3);

--//draw hue selector
draw_sprite_stretched(global.PE.shue, 0, self.ox, self.offsetHueY, self.wh + 1, self.HueH);
draw_rectangle(self.ox, self.offsetHueY, self.offsetW, self.offsetHueY + self.HueH - 1, true);
draw_sprite_stretched(scursor, 1, HueX - 1.5, HueY, 3, 5);

--//draw alpha selector
draw_sprite_stretched(global.PE.salpha, 0, self.ox, self.offsetAlphaY, self.wh + 1, self.AlphaH);
gpu_set_blendmode(bm_add);
draw_rectangle_color(self.ox, self.offsetAlphaY, self.offsetW, self.offsetAlphaY + self.AlphaH - 1, c_black, col, col, c_black, false);
gpu_set_blendmode(bm_normal);
draw_rectangle(self.ox, self.offsetAlphaY, self.offsetW, self.offsetAlphaY + self.AlphaH - 1, true);
draw_sprite_stretched(scursor, 1, AlphaX - 1.5, AlphaY, 3, 5);

--//draw text with color values and currently selected part of the palette
let part_name = ds_grid_get(global.PE.part_names_grid, self.mode, self.selectedPart);
let txtScale = 0.75;
let strW = string_width(part_name) * txtScale;
if (strW > self.wh - 1){
    txtScale *= (self.wh - 1) / strW;}
draw_set_valign(fa_middle);
draw_text_transformed(self.offsetPartNameX, self.offsetPartNameY, part_name, txtScale, txtScale, 0);
draw_set_valign(fa_top);
draw_text(self.ox, self.offsetTextY, string("H:{0}\nS:{1}\nV:{2}\nHEX:{3}\nAlpha:{4}", round(self.h), round(self.s), round(self.v), self.hex, self.a));
draw_text(self.ox + 34, self.offsetTextY, string("R:{0}\nG:{1}\nB:{2}", self.R, self.G, self.B));

--//draw rectangle with current color
draw_sprite(global.PE.scolor_rect, 0, self.ox, self.offsetColRectY);
draw_set_alpha(self.a)
draw_rectangle_color(self.ox, self.offsetColRectY, self.ox + self.colRectWH, self.offsetColRectY + self.colRectWH, col, col, col, col, false);
draw_set_alpha(1);
col = make_colour_rgb(255 - self.R, 255 - self.G, 255 - self.B);
draw_rectangle_color(self.ox, self.offsetColRectY, self.ox + self.colRectWH, self.offsetColRectY + self.colRectWH, col, col, col, col, true);

--//draw palette color selector
draw_sprite(global.PE.spalette_bg, 0, self.ox - 9, self.oy);
draw_sprite_part_ext(global.PE.splayer_palette[self.mode],0, 1,0, 1,56, self.ox - 9,self.oy, 4,4, c_white, 1);
let selectedPartY = self.oy + (self.selectedPart << 2);
let selectedPartX = self.ox - 9;
let selectedPartWH = 3;
if(self.selectedPart > 55)
{
    selectedPartY = self.offsetHeadY + (self.selectedPart - 56) * 10;
    selectedPartX = self.offsetHeadX - 1 + (57 - self.selectedPart) * 2;
    selectedPartWH = 10;
}
else{
    draw_rectangle_color(selectedPartX, selectedPartY, selectedPartX + selectedPartWH, selectedPartY + selectedPartWH, col, col, col, col, true); }
draw_sprite_stretched(sui_arrow, 0, selectedPartX - 3, selectedPartY - 1, 4, 6);

if (self.mode != 1 or global.PE.forceHair)
{
    --//draw head and hair selector
    let hair_start_col = if (!global.ameli_mode_) {global.hair_start_col_} else {global.PE.palette_data[2][55]};
    let hair_end_col = if (!global.ameli_mode_) {merge_color(global.hair_start_col_, global.hair_end_col_, 0.667)} else {hair_start_col};
    let surf = surface_create(self.headW, self.headH);
    surface_set_target(surf);

    draw_rectangle_color(0, 0, self.headW - 1, self.headH - 1, hair_start_col, hair_start_col, hair_end_col, hair_end_col, false);
    gpu_set_blendmode(bm_subtract);
    draw_sprite(global.PE.splayer_head, 0, 0, 0);
    gpu_set_blendmode(bm_normal);

    shader_replace_simple_set_hook(global.player_palette_shader_);
    texture_set_stage(global.main_shader_palette_pointer, global.palette_texture);
    draw_sprite(global.PE.splayer_head, 1, 0, 0);
    shader_replace_simple_reset_hook();

    surface_reset_target();
    draw_surface(surf, self.offsetHeadX, self.offsetHeadY);
    surface_free(surf);

    --//draw hair length buttons
    draw_set_halign(fa_center);
    draw_text(self.offsetHairButtonsX + self.HairButtonsWH, self.offsetHairButtonsY - 12, string(global.PE.hair_number[self.mode]));
    draw_set_halign(fa_left);
    let buttonPressed = self.pressTimer > 0;
    let bDownSpr = match ((self.isHoveringHairLength + buttonPressed) * (self.hairLengthButton > 0))
    {
        case 2 {s_scroll_bar_down_button_pressed}
        case 1 {s_scroll_bar_down_button_hover}
        else   {s_scroll_bar_down_button}
    }
    let bUpSpr = match ((self.isHoveringHairLength + buttonPressed) * (self.hairLengthButton < 0))
    {
        case 2 {s_scroll_bar_up_button_pressed}
        case 1 {s_scroll_bar_up_button_hover}
        else   {s_scroll_bar_up_button}
    }
    draw_sprite(bDownSpr, 0, self.offsetHairButtonsX, self.offsetHairButtonsY);
    draw_sprite(bUpSpr, 0, self.offsetHairButtonsX + self.HairButtonsWH, self.offsetHairButtonsY);
    if (buttonPressed) {
        self.pressTimer -= 1;}
}

--//draw preview
shader_replace_simple_set_hook(global.player_palette_shader_);
texture_set_stage(global.main_shader_palette_pointer, global.palette_texture);

let previewSize = 48;
draw_sprite_stretched(global.PE.splayer_palette_ref[self.mode], 1, self.offsetPreviewX, self.offsetPreviewY, previewSize, previewSize);

shader_replace_simple_reset_hook();

draw_set_color(c_white);

--//draw cursor when hovering
if (!self.isHovering){
    return; }

if (self.isHoveringSatVal or self.isHoveringHue or self.isHoveringAlpha){
    draw_sprite(scursor, 1, global.mouse_gui_x_, global.mouse_gui_y_); }
else{
    draw_cursor(); }

```

## alarm_0
```
self.color = make_colour_hsv(self.h, self.s, self.v) | ((floor(self.a * 255) << 24));
self.R = colour_get_red(self.color);
self.G = colour_get_green(self.color);
self.B = colour_get_blue(self.color);

self.generateHex();

match (self.selectedPart) 
{
    case 56 {global.hair_start_col_ = self.color} 
    case 57 {global.hair_end_col_ = self.color}
    else    {global.PE.palette_data[self.mode][self.selectedPart] = self.color}
}

with(oplayer)
{
    self.hair_start_col = global.hair_start_col_;
    self.hair_end_col = global.hair_end_col_;
}

let surf = surface_create(2, global.PE.paletteH);
surface_set_target(surf);
draw_clear_alpha(c_black, 0);
draw_sprite(global.PE.splayer_palette[self.mode], 0, 0, 0);
let i = 0;
gpu_set_blendmode_ext( bm_one, bm_zero );
while (i < global.PE.paletteH)
{
    draw_set_alpha((global.PE.palette_data[self.mode][i] >> 24) * self.inv255);
    draw_point_colour(1, i, global.PE.palette_data[self.mode][i]);
    draw_set_alpha(1);
    i += 1;
}
gpu_set_blendmode(bm_normal);
surface_reset_target();
sprite_delete(global.PE.splayer_palette[self.mode]);
global.PE.splayer_palette[self.mode] = sprite_create_from_surface(surf, 0, 0, 2, 56, false, false, 0, 0);
surface_free(surf);

shader_replace_simple_set_hook(global.player_palette_shader_);
global.palette_texture = sprite_get_texture(global.PE.splayer_palette[self.mode], 0);
texture_set_stage(global.main_shader_palette_pointer, global.palette_texture);
shader_replace_simple_reset_hook();
if(!global.PE.turnOffPlayerReplace)
{
    global.__scarf_charged = global.PE.palette_data[1][54];
    global.__scarf_uncharged = global.PE.palette_data[1][55];
    global.__ameli_hair = global.PE.palette_data[2][55];
}
if  (global.ameli_mode_)
{
    global.hair_start_col_ = global.PE.palette_data[2][55];
    global.hair_end_col_ = global.PE.palette_data[2][55];
}
```

## cleanup
```
let modeStr = match(self.mode)
{
    case 2 {"_ameli"}
    case 1 {"_maya"}
    else   {"_fern"}
}
sprite_save(global.PE.splayer_palette[self.mode], 0, global.PE.modPath + "saved_palettes/palette" + modeStr + ".png");
if (global.PE.turnOffPlayerReplace)
{
    if (global.maya_mode)
    {
        sprite_save(global.PE.splayer_palette[self.mode], 0, "mods/rmml/maya_palette/palette.png");
    }
    if (global.ameli_mode_)
    {
        sprite_save(global.PE.splayer_palette[self.mode], 0, "mods/rmml/ameli_palette/palette.png");
    }
}

ini_open(global.PE.modPath + "saved_palettes/hair_data.ini");
ini_write_real("hair_data", "length", global.PE.hair_number[0]);
ini_write_real("hair_data", "maya_length", global.PE.hair_number[1]);
ini_write_real("hair_data", "ameli_length", global.PE.hair_number[2]);
ini_write_real("hair_data", "start_col", global.hair_start_col_);
ini_write_real("hair_data", "end_col", global.hair_end_col_);
ini_close();

let surf = surface_create(64, 64);
surface_set_target(surf);
draw_clear_alpha(c_black, 0);
shader_replace_simple_set_hook(global.player_palette_shader_);
texture_set_stage(global.main_shader_palette_pointer, global.palette_texture);

draw_sprite(global.PE.splayer_palette_ref[self.mode], 1, 0, 0);

if (global.PE.turnOffPlayerReplace and self.mode != 0){
    shader_set_uniform_f_array(shader_get_uniform(global.player_palette_shader_, "palette_uvs"), [0,0, 0,0]);}

shader_replace_simple_reset_hook();

surface_reset_target();
surface_save(surf, global.PE.modPath + "saved_palettes/palette_ref" + modeStr + ".png");
surface_free(surf);


```

# player
## create
```
event_inherited();
```
## alarm_0
```
event_inherited();
```
## step
```
event_inherited();
```
## step_end
```
event_inherited();
```
## animation_end
```
event_inherited();
```
## draw
```
let check = self.draw_type != 1 and self.sprite_index != self.turnaround_sprite and global.ameli_mode_;
if (check)
{
    hair_code();
}
else
{
    self.hair_x = lerp(self.hair_x, self.head_x, 0.6);
    self.hair_y = lerp(self.hair_y, self.head_y + 3, 0.6);
    hair_code();
}
self.start_palette();
event_inherited();
shader_replace_simple_reset_hook();
if (self.sprite_index == self.turnaround_sprite)
{
    self.hair_y = self.y - 31;
    self.hair_x = self.x - (7 * self.draw_xscale);
}
if (check and self.hair != -1)
{
    draw_circle_color(self.hair[0].x, self.hair[0].y, self.hair_size, global.__ameli_hair, global.__ameli_hair, 0);
}
```
## draw_end
```
event_inherited();
```
## draw_gui_begin
```
event_inherited();
```
## cleanup
```
event_inherited();
```