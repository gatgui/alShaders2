import sys
import excons
import os
import imp
import re
from excons.tools import arnold


env = excons.MakeBaseEnv()
out_basedir = excons.OutputBaseDirectory()


if 5 > arnold.Version(asString=False)[0]:
    print "alshaders requires at least Arnold 5.0.0.1"
    sys.exit(1)


def _checkDir(path):
    if not os.path.isdir(os.path.dirname(path)):
        os.makedirs(os.path.dirname(path))


def uiGen(target, source, env):
    need_to_update_mtd = False
    need_to_update_template = False
    ui_file = os.path.abspath(source[0].get_path())
    shader_name = os.path.splitext(os.path.basename(ui_file))[0]
    mtd_path = os.path.abspath(os.path.join(out_basedir, "arnold", "%s.mtd") % shader_name)
    template_path = os.path.abspath(os.path.join(out_basedir, "maya", "%sTemplate.py") % shader_name)

    if not os.path.isfile(mtd_path) or os.stat(ui_file).st_mtime > os.stat(mtd_path):
        need_to_update_mtd = True
    if not os.path.isfile(template_path) or os.stat(ui_file).st_mtime > os.stat(template_path):
        need_to_update_template = True

    if not need_to_update_mtd and not need_to_update_template:
        return None

    ui_gen = imp.load_source("uigen", os.path.abspath("uigen.py"))    
    ui = ui_gen.ShaderDef()
    globals_dict = {'ui': ui}
    execfile(ui_file, globals_dict)

    _checkDir(mtd_path)
    _checkDir(template_path)

    if need_to_update_mtd:
        print "> generate %s mtd" % shader_name
        ui_gen.WriteMTD(ui, mtd_path)

    if need_to_update_template:
        print "> generate %s template" % shader_name
        ui_gen.WriteAETemplate(ui, template_path)

        lines = []
        with open(template_path, "r") as f:
            lines = f.readlines()
        with open(template_path, "w") as f:
            for l in lines:
                res = re.match("from\salShaders\simport\s[*]", l)
                if res:
                    with open("common/alShaders.py", "r") as af:
                        f.write(af.read())
                else:
                    f.write(l)

    return None


prjs = []
cppflags = ""
crypto_install = {}

if sys.platform != "win32":
    cppflags += " -Wno-unused-parameter"
    cppflags += " -Wno-sign-compare"
    cppflags += " -std=c++11"


## cryptomatte
prjs.append({"name": "cryptomatte",
             "alias": "cryptomatte-shader",
             "prefix": "arnold",
             "type": "dynamicmodule",
             "cppflags": cppflags,
             "incdirs": [],
             "ext": arnold.PluginExt(),
             "libs": [],
             "symvis": "hidden",
             "srcs": excons.Glob("cryptomatte/*.cpp"),
             "install": crypto_install,
             "custom": [arnold.Require]})
 

env["BUILDERS"]["uiBuild"] = Builder(action=uiGen)
crypto_ui = env.uiBuild(None, ["cryptomatte/cryptomatte.ui"])

env.Alias("cryptomatte-ui", crypto_ui)
env.Alias("cryptomatte", ["cryptomatte-shader", "cryptomatte-ui"])


## declare build targets
excons.DeclareTargets(env, prjs)
