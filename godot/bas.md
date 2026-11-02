# 座標軸の表示

bas.gd
```gdscript:bas.gd
extends MeshInstance3D

func _ready() -> void:
	mesh = ImmediateMesh.new()
	
	var mat = StandardMaterial3D.new()
	mat.shading_mode = BaseMaterial3D.SHADING_MODE_UNSHADED
	mat.vertex_color_use_as_albedo = true
	mat.no_depth_test = true
	
	material_override = mat

func _process(delta: float) -> void:
	var imm: ImmediateMesh = mesh
	imm.clear_surfaces()
	
	var parent: Node3D = get_parent()
	if not parent: return
	
	var bas: Basis = parent.transform.basis
	
	imm.surface_begin(Mesh.PRIMITIVE_LINES)

  # x
	imm.surface_set_color(Color.RED)
	imm.surface_add_vertex(Vector3.ZERO)
	imm.surface_add_vertex(bas.x)

  # y
	imm.surface_set_color(Color.GREEN)
	imm.surface_add_vertex(Vector3.ZERO)
	imm.surface_add_vertex(bas.y)

  # z
	imm.surface_set_color(Color.BLUE)
	imm.surface_add_vertex(Vector3.ZERO)
	imm.surface_add_vertex(bas.z)
	
	imm.surface_end()
```
