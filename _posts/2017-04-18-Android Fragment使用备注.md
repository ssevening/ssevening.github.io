---
title: Android Fragment使用备注
category: Android开发
feature_image: "https://ssevening.github.io/assets/android.png"
image: "https://ssevening.github.io/assets/android.png"
---

在日常工作中，到底什么时候获取参数，什么时候初始化接口，还是在这里记一下。

<!-- more -->

# 一、Fragment的版本兼容
* 默认一系列的都不推荐使用默认的Fragment
* 而是推荐使用Support包中的。所以后有下面的替换Fragment的代码。

```
				FragmentManager fragmentManager = getSupportFragmentManager();
                FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();

                Fragment fragment = (Fragment) ARouter.getInstance().build("/fragment/productFragment").navigation();
                Bundle bundle = new Bundle();
                bundle.putString("productId", "1234567890");
                fragment.setArguments(bundle);
                fragmentTransaction.replace(R.id.rl_container, fragment);
                fragmentTransaction.addToBackStack(null);
                fragmentTransaction.commit();
                
```
* 注意，一定是getSupportFragmentManager();

# 二、什么时候初始化Fragment中的声明的接口

Google官方源码中，**是写在onAttach中的，项目中，通常会写在onActivityCreated。**
详见下面Google的源码

	public static class FragmentA extends ListFragment {
	                    OnArticleSelectedListener mListener;
	    ...
	                    @Override
	                    public void onAttach(Activity activity) {
	                        super.onAttach(activity);
	                        try {
	                            mListener = (OnArticleSelectedListener) activity;
	                        } catch (ClassCastException e) {
	                            throw new ClassCastException(activity.toString() + " must implement OnArticleSelectedListener");
	                        }
	                    }
	    ...
	                }


# 三、什么时候可以获取Argus
* **onCreateView 就可以直接获取到Args.**

		if (getArguments() != null) {
            String productId = getArguments().getString("productId");
            Toast.makeText(getActivity(), "productId is " + productId, Toast.LENGTH_SHORT).show();
        }


所以后面，要注意此类Fragment的使用。


欢迎关注作者微信公众号，及时获得作者更新：

![微信公众号](https://ssevening.github.io/assets/weichat_qrcode.jpg)

另外还建立了小密圈：圈主 和 嘉宾 都就职于 阿里巴巴 的顶尖开发者，开发的app被Google 编辑推荐，对性能，架构，图片，MD设计都有研究和深入，欢迎大家加入，提升自己，一起进步，互相帮助交流！

![微信公众号](https://ssevening.github.io/assets/mi_qrcode.png)


