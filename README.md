# wzmao.github.io

## Here is the personal website for Wenzhi Mao.
---
## Here is the personal website for Wenzhi Mao1.

```python
class Feature(object):
	"""docstring for Feature"""
	def __init__(self, name, print_name = None):
		super(Feature, self).__init__()
		self.name = name
		self.print_name = self.print_name if print_name is not None else self.name
	def __repr__(self):
		return '{}<{}>'.format(type(self).__name__, self.print_name)
	def __eq__(self, other):
		if isinstance(other, Feature):
			return (self.name == other.name)
		if isinstance(other, str):
			return self.name == other
		return False

class NormalFeature(Feature):
	"""docstring for NormalFeature"""
	def __init__(self, candidate_var, *arg, **kwargs):
		super(NormalFeature, self).__init__(*arg, **kwargs)
		self.candidate_var = candidate_var
	def toScreenStr(self, val_dict):
		if self.name in val_dict:
			return '{} = {}'.format(self.print_name, val_dict[self.name])
		return None
	def toPredStr(self, val_dict):
		if self.name in val_dict:
			return '{}\x03{}'.format(self.name, val_dict[self.name])
		return None
	def debugSampleGenerate(self, sample):
		l=[]
		v=[]
		for var in self.candidate_var:
			newsample = sample.copy()
			newsample[self.name]=var
			l.append(newsample)
		newsample = sample.copy()
		newsample.remove(self.name)
		l.append(newsample)
		return [self.name],self.candidate_var[:]+[None],[l]

class OneHotFeature(Feature):
	"""docstring for OneHotFeature"""
	def __init__(self, namelist = None, *arg, **kwargs):
		super(OneHotFeature, self).__init__(*arg, **kwargs)
		self.namelist = [str(i) for i in namelist]
	def toScreenStr(self, val_dict):
		if self.name in val_dict:
			val = val_dict[self.name]
			if isinstance(val,int):
				if 0<=val<len(self.namelist):
					return '{} = {}'.format(self.print_name, self.namelist[val])
				else:
					return '{} = {}'.format(self.print_name, "No match")
			return None
		return None
	def toPredStr(self, val_dict):
		if self.name in val_dict:
			val=int(val_dict[self.name])
			return '\x02'.join(['{}_{}\x03{}'.format(self.name, i, 1 if val == i else 0) for i in range(len(self.namelist))])
		return None
	def debugSampleGenerate(self, sample):
		l=[]
		for var in range(len(self.namelist)):
			newsample = sample.copy()
			newsample[self.name] = var
			l.append(newsample)
		newsample = sample.copy()
		newsample[self.name] = -1
		l.append(newsample)
		newsample = sample.copy()
		newsample.remove(self.name)
		l.append(newsample)
		return [self.name],list(range(len(self.namelist)))+[-1,None],[l]

class DocTypeFeature(Feature):
	"""docstring for DocTypeFeature"""
	def __init__(self, debug_type_list = None, vars_list = None, *arg, **kwargs):
		kwargs['name'] = 'doc_type'
		super(DocTypeFeature, self).__init__(*arg, **kwargs)
		self.debug_type_list = debug_type_list if debug_type_list is not None else []
		self.debug_type_list = [i if i.startswith('doc_type_') else 'doc_type_{}'.format(i) for i in self.debug_type_list]
		self.vars_list = [0,.1,.2,.3,.4,.5,.6,.7,.8,.9,1.0][::5] if vars_list is None else vars_list
	def toScreenStr(self, val_dict):
		tojoin = ['{} = {}'.format(i, val_dict[i]) for i in val_dict if i.startswith("doc_type_")]
		if tojoin:
			return '\t'.join(tojoin+['doc_type = {}'.format(len(tojoin))])
		return None
	def toPredStr(self, val_dict):
		tojoin = ['{}\x03{}'.format(i, val_dict[i]) for i in val_dict if i.startswith("doc_type_")]
		if tojoin:
			return '\x02'.join(tojoin+['doc_type\x03{}'.format(len(tojoin))])
		return None
	def debugSampleGenerate(self, sample):
		ll=[]
		vv=[]
		for feature in self.debug_type_list:
			l=[]
			for var in self.vars_list:
				newsample = sample.copy()
				newsample[feature] =  var
				l.append(newsample)
			newsample = sample.copy()
			newsample.remove(feature)
			l.append(newsample)
			ll.append(l)
			vv.append(self.vars_list[:]+[None])
		return self.debug_type_list[:],vv,ll

class Sample(dict):
	"""docstring for Sample"""
	def __init__(self, features = None):
		super(Sample, self).__init__(features if features is not None else [])
	def copy(self):
		return Sample(self)
	def remove(self, name):
		if name in self:
			self.pop(name)
	def toPredStr(self, feature_list):
		tojoin = [f.toPredStr(self) for f in feature_list]
		tojoin = [i for i in tojoin if i is not None]
		return '\x02'.join(tojoin)
	def toScreenStr(self, feature_list):
		tojoin = [f.toScreenStr(self) for f in feature_list]
		tojoin = [i for i in tojoin if i is not None]
		return '\t'.join(tojoin)


sample = Sample()

sample['feature_0'] = 1
sample.update({"feature_1":2, 'feature_2':0.5,'doc_type_5':3})
sample.remove('feature_011')

print(sample.toPredStr([NormalFeature(name = 'feature_0', candidate_var = [0,1,2])]))
print(sample.toPredStr([OneHotFeature(name = 'feature_1', namelist = ['type_1','type_2','type_3','type_4'])]))
print(sample.toPredStr([DocTypeFeature(debug_type_list = ['1','5','7'])]))
print()

print(sample.toScreenStr([NormalFeature(name = 'feature_0', candidate_var = [0,1,2])]))
print(sample.toScreenStr([OneHotFeature(name = 'feature_1', namelist = ['type_1','type_2','type_3','type_4'])]))
print(sample.toScreenStr([DocTypeFeature(debug_type_list = ['1','5','7'])]))
print()

print(NormalFeature(name = 'feature_0', candidate_var = [0,1,2]).debugSampleGenerate(sample))
print(OneHotFeature(name = 'feature_1', namelist = ['type_1','type_2','type_3','type_4']).debugSampleGenerate(sample))
print(DocTypeFeature(debug_type_list = ['1','5','7']).debugSampleGenerate(sample))
```
